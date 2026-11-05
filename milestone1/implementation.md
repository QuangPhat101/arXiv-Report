# Implementation

## System architecture

The crawler codebase is organized into modules with distinct responsibilities across discovery, downloading, extraction, citation processing, and performance profiling:

- `src/crawler.py`: Orchestrates the data acquisition workflow. It supports both the baseline single-threaded pipeline (`run_sequential_pipeline`) and the concurrent 3-stage Producer-Consumer pipeline (`run_parallel_pipeline`). It handles command-line arguments, bounded queue dispatching, two-tier checkpoint resumption, and process synchronization.
- `src/discovery.py`: Interfaces with the official arXiv API Atom feed using `arxiv.py` (version 4.x) to query paper metadata (title, authors, submission date, revision dates, venue) and detect all available revision identifiers (from `v1` to `vN`) through `discover_papers_batch` and `discover_paper`.
- `src/downloader.py`: Retrieves raw source packages from the official arXiv export endpoint (`https://export.arxiv.org/src/{versioned_id}`). It implements chunked HTTP streaming, temporary file buffering (`.part`), and HTTP Range header resumption (`Range: bytes={existing_bytes}-`) on HTTP status 206.
- `src/extractor.py`: Inspects archive payloads, validates archive member paths to prevent directory traversal vulnerabilities (Zip Slip), and extracts only `.tex` and `.bib` files into version directories. It identifies and handles multi-file tarballs, single-file gzip archives, and PDF-only submissions.
- `src/references.py`: Interfaces with the Semantic Scholar Graph API to crawl citation graphs, bibliographic records, and external arXiv identifiers. It synchronizes rate limits across threads using `_s2_lock` and applies exponential backoff with randomized jitter on HTTP status 429.
- `src/utils.py`: Supplies shared cross-cutting utilities, including thread-safe monotonic rate limiters (`RateLimiter`), an exponential backoff retry decorator (`retry_with_backoff`), and structured logging (`setup_logger`). Singleton rate limiters enforce minimum intervals for arXiv and Semantic Scholar endpoints.
- `src/profiler.py`: The `CrawlerProfiler` monitors process resource metrics (resident set size memory via `psutil`), per-paper phase durations, and error classifications. Telemetry recording is synchronized via `_records_lock`, and metrics are exported to JSON for benchmark analysis.
- `tests/test_parallel_crawler.py`: Unit test suite verifying the correctness of thread synchronization, CLI parsing, pipeline error isolation, and end-to-end Producer-Consumer execution.

## Pipeline architecture and execution flow

The system provides two execution engines: a baseline sequential engine for controlled benchmarking and a 3-stage Producer-Consumer parallel pipeline for high-throughput crawling.

```text
[ Stage 1: Discovery Producer ]
            |
            | download_queue (Queue, maxsize=20, PaperDownloadJob)
            v
[ Stage 2: Download Workers Pool ]
            |
            | extract_s2_queue (Queue, maxsize=10, PaperProcessJob)
            v
[ Stage 3: Extract + S2 Workers Pool ]
```

### The 3-stage producer-consumer pipeline

The parallel engine partitions work across decoupled thread pools connected by bounded queues to overlap network I/O, local disk extraction, and external API requests:

1. Stage 1 (Discovery producer):
   - The producer checks local disk caches for existing `metadata.json` files.
   - Pending papers lacking metadata are queried in batches (chunks of size `batch_size`, default 100) using `discover_papers_batch` to minimize HTTP round trips.
   - For each paper, it constructs a `PaperDownloadJob` containing the paper ID and metadata, then places it into `download_queue` (bounded capacity 20).
   - Once all targets are queued, it injects sentinel `None` tokens to signal worker termination.

2. Stage 2 (Download workers pool):
   - A pool of download worker threads (default 3 workers) pulls `PaperDownloadJob` items from `download_queue`.
   - Each worker maintains its own persistent `requests.Session` for HTTP connection pooling.
   - For each revision, the worker calls `download_source` with `allow_resume=True` and a 60.0-second network timeout, governed by the shared `arxiv_rate_limiter`.
   - If an ongoing download is interrupted by network instability, partial content is saved in a `.part` temporary file. The next retry sends an HTTP `Range` request to fetch only remaining bytes (HTTP status 206 Partial Content).
   - Downloaded archive paths and elapsed download timings are packaged into a `PaperProcessJob` and pushed to `extract_s2_queue` (bounded capacity 10).

3. Stage 3 (Extract and S2 workers pool):
   - A secondary worker pool (default `min(download_workers, 4)`) consumes `PaperProcessJob` objects from `extract_s2_queue`.
   - The worker calls `extract_tex_bib` to decompress `.tex` and `.bib` files directly into version folders (`v1/`, `v2/`). Media assets, image files, and compiled binaries are discarded, and temporary archive files are unlinked immediately after extraction. If the source package is absent (HTTP 404 from arXiv), the version is recorded as `PDF_ONLY`.
   - If extraction succeeds, the worker queries `references.get_references` for citation metadata. Concurrent requests are rate-limited, and rate-limit backoffs apply randomized jitter.
   - Under mutex protection (`state_lock`), successful papers are appended atomically to `checkpoint.txt`. Telemetry data is recorded in the profiler under `profiler_lock`, and progress lines are printed safely using `print_lock`.

### Two-tier checkpoint and resumption

State is tracked at two granular levels:

1. Version level: Each successfully unpacked version directory receives an atomic `.complete` stamp file containing extraction status and byte sizes (`status|raw_bytes|extracted_bytes`).
2. Paper level: When all versions and citation records for a publication are verified by `is_paper_complete`, the paper ID is committed atomically to `checkpoint.txt`. Subsequent runs check `checkpoint.txt` to bypass completed papers without redundant requests.

## Tools and libraries

The crawler uses standard Python libraries alongside focused third-party packages:

- `arxiv.py (>= 4.0.0)`: Communicates with the arXiv API Atom feed using the `client.results(search)` interface for batch querying publication metadata and detecting version suffixes.
- `requests`: Manages HTTP transport for source downloads and citation queries, utilizing connection pooling (`requests.Session`), streaming (`stream=True`), and range headers.
- `tarfile` and `gzip`: Handle archive unpacking for multi-file tarballs and single-file compressed LaTeX documents, with path validation against directory traversal attacks.
- `psutil`: Collects runtime system telemetry, sampling resident set size (RSS) memory periodically to capture process memory profiles.
- `argparse` and `pathlib`: Parse configuration options, resolve full arXiv identifiers, and manage cross-platform path operations.
- `pytest`: Executes unit test suites to validate pipeline concurrency, rate-limiter thread safety, and edge-case recovery.

## Rate limiting and concurrency control

Because the crawler queries multiple external endpoints concurrently, strict rate limiting and synchronization mechanisms are enforced:

- The `RateLimiter` class in `utils.py` tracks timestamps using `time.monotonic()` inside a `threading.Lock()` critical section. When a worker invokes `wait()`, elapsed time since the previous request is computed, and the thread sleeps only for the remaining interval before updating `last_request_time`. A shared instance (`arxiv_rate_limiter`) maintains a 3.0-second delay across all download workers, and `s2_rate_limiter` maintains a 1.0-second delay with an API key or 2.5 seconds without.
- In `downloader.py`, stream interruptions preserve partially downloaded bytes in a `.part` file. Upon retry with `allow_resume=True`, the request sets `Range: bytes={existing_bytes}-`. When the server returns HTTP 206, the file stream opens in append mode (`ab`), avoiding full re-transmissions.
- When Semantic Scholar responds with HTTP 429, `references.py` synchronizes the cooldown window using `_s2_lock`. In parallel mode, backoff incorporates random jitter: `cooldown = min(30.0, base_cooldown * (1.3 ** (attempt - 1)) + random.uniform(-0.5, 0.5))`. This jitter desynchronizes retries across workers, preventing thundering herd collisions.

## Execution modes and benchmark instrumentation

The crawler CLI exposes flags to support both production crawling and controlled empirical benchmarks:

```bash
python src/crawler.py --month 2505 --start 10666 --end 15665 --output ./data --parallel
```

- `--parallel`: Activates the 3-stage Producer-Consumer pipeline, setting default worker counts to 3 download threads.
- `--baseline`: Forces sequential execution with single-worker processing, 30.0-second network timeouts, 5 Semantic Scholar retries without jitter, and disabled Range resumption.
- `--n-workers`, `--download-workers`, `--s2-workers`: Allow fine-grained allocation of worker threads across download and extraction stages.
- `--profile` and `--stats-file`: Activate `CrawlerProfiler` to track memory RSS samples, measure phase durations (`discovery`, `download`, `extract`, `s2_references`, `rate_limit_sleep`), aggregate error classifications, and export structured benchmark summaries.

## Edge cases and failure recovery

The crawler handles irregular submissions, malformed archives, and transient network conditions. The table below summarizes conditions, implementation strategies, and observed frequencies across the validation sample of 5,000 papers (8,205 versions) in the range `2503.01914` to `2503.06913`:

| Edge case | Handling strategy | Observed frequency |
| :--- | :--- | :--- |
| PDF-only submission (no source archive on arXiv) | `downloader` returns `None` on HTTP 404; the crawler records the version as `PDF_ONLY` with an atomic `.complete` stamp | 24 versions |
| Archive without `.tex` or `.bib` files (figures or standalone PDF only) | `extractor` records `PDF_ONLY` when an archive contains no LaTeX files | 590 versions |
| Single-file gzip archive (one `.tex` file, not a tarball) | Decompressed directly with `gzip` and stored as `main.tex` | 816 versions |
| Raw PDF or PostScript binary payload | Identified via magic bytes (`%PDF-`, `%!PS`) and classified as `PDF_ONLY` | Included in counts above |
| Path traversal vulnerabilities in archive members (Zip Slip) | Destination paths are verified against target directories; path escapes, symlinks, and device nodes are skipped | 0 malicious attempts |
| Multiple versions per publication | Revisions `v1` to `vN` are detected from metadata and processed through worker pools | Maximum 26 versions; 43% of papers contain multiple versions |
| Paper not indexed in Semantic Scholar | Explicit HTTP 404 responses write an empty array to `references.json`, allowing the paper to complete | 270 papers (5.4%) |
| Rate limiting encounters (HTTP 429) | Monotonic rate limiters, mutex-protected cooldowns, and jittered exponential backoff retries | 277 events in the original sequential crawl, all recovered |
| Interrupted network streams | Partial payloads are preserved in `.part` files and resumed via HTTP Range requests (HTTP 206) | Not quantified |
| Transient server and network errors (5xx, timeouts) | Retried using exponential backoff; unresolvable failures raise exceptions so the paper is retried on the next pass | Not quantified |
| Residual failures after retry limits | Unfinished papers are logged with error reasons; a subsequent pass targets pending items from checkpoint | 2 papers, recovered in round 2 |
| Process interruptions | Two-tier checkpointing (`checkpoint.txt` and `.complete` stamps) bypasses finished work upon restart | Not quantified |
| Cross-month ID boundaries | The `--start-month-max` flag limits ID generation to actual monthly submissions, avoiding non-existent ranges | Not applicable (the sample lies within a single month) |

### Error handling integrity

The crawler records an empty citation list only when Semantic Scholar returns an explicit HTTP 404. Other failures raise an exception so the paper remains pending for retry on the next pass.

### Storage optimization from selective extraction

Across all 8,205 versions in the observed dataset, downloaded raw archives occupied approximately 38.7 GB of storage. By selectively retaining only `.tex` and `.bib` files, the extracted corpus requires only 2.7 GB, representing a storage reduction of approximately 93%. Original LaTeX syntax remains completely unmodified, preserving `\includegraphics` commands and `figure` environments for downstream document parsing in Milestone 2.
