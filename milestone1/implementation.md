# Implementation

<!-- Phát viết phần này -->

## Kiến trúc hệ thống

<!-- Mô tả cấu trúc module: crawler.py, discovery.py, downloader.py, extractor.py, references.py, utils.py -->

## Pipeline Flow

<!-- Mô tả flow: Generate IDs → Discovery → Download → Extract → References -->
<!-- Giải thích checkpoint/resume -->

## Tools & Libraries

<!-- Liệt kê và giải thích tại sao dùng: arxiv.py v4.x, requests, Semantic Scholar API, tarfile, ... -->

## Rate Limiting & Retry

<!-- Giải thích rate limiter (3s arXiv, 1s S2), exponential backoff, xử lý 429/403 -->

## Edge Cases

The pipeline must survive the many irregular shapes an arXiv submission can take, as well as unreliable network conditions. Each case below lists the symptom, how the crawler handles it, and how often it occurred in the range `2503.01914`–`2503.06913` (5,000 papers, 8,205 versions).

| Edge case | Handling | Observed |
|:---|:---|:---|
| PDF-only submission (no source on arXiv) | `downloader` returns `None` on HTTP 404, `extractor` returns `PDF_ONLY` and writes an empty version folder with a `.complete` stamp | 24 versions |
| Archive without any `.tex`/`.bib` (figures or PDF only) | `extractor` returns `PDF_ONLY` instead of `FAILED` | 590 versions |
| Single-file gzip (one `.tex`, not a tarball) | Decompressed with `gzip` and saved byte-for-byte as `main.tex` | 816 versions |
| Raw PDF or PostScript payload | Detected by magic bytes (`%PDF-`, `%!PS`) and treated as `PDF_ONLY` | included above |
| Path traversal in tar members (Zip Slip) | Every member path is resolved and skipped if it escapes the version folder; symlinks, hardlinks and devices are never written | 0 attempts |
| Many versions per paper | All versions `v1`…`vN` are discovered from the latest `entry_id` and downloaded | max 26 versions; 43% of papers have more than one |
| Paper not indexed by Semantic Scholar | HTTP 404 is saved as an empty `references.json` so the paper still completes | 270 papers (5.4%) |
| Rate limiting (HTTP 429) | Shared monotonic rate limiters (3 s arXiv, 1 s Semantic Scholar), `Retry-After` cooldown and exponential backoff capped at 30 s, up to 10 attempts | 277 events, all recovered |
| Transient server or network errors (5xx, timeouts) | Retried with exponential backoff; on exhaustion the error is raised rather than writing empty data | — |
| Papers still failing after all retries | After each round the crawler automatically re-runs only the failed papers until every one succeeds | 2 papers, recovered in round 2 |
| Interrupted runs | Two-level checkpoint (`checkpoint.txt` per paper, `.complete` per version) lets a restart skip finished work | — |
| Cross-month ID ranges | `--start-month-max` caps the first month so no non-existent IDs are generated | not applicable to this range |

### Why empty results are never written on errors

A subtle failure mode is treating a transient error as "no data". If a 429 or timeout from Semantic Scholar were saved as `[]`, the paper would be marked complete and its references silently lost. The crawler therefore only writes an empty list for a definitive HTTP 404, and raises on any other failure so the paper is retried in the next round.

### Effect of keeping only `.tex` and `.bib`

Across all 8,205 versions, the downloaded archives totalled about 38.7 GB, while the extracted `.tex`/`.bib` files take about 2.7 GB, a reduction of roughly 93%. LaTeX content is copied unchanged, so `\includegraphics` commands and `figure` environments are preserved even though the image files are dropped.
