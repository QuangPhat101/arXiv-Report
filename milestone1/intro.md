# Milestone 1: Data Crawling

## Project overview

This project is part of the Introduction to Data Science course at VNUHCM-US (HCMUS). The project spans three milestones:
1. Milestone 1 (Data Crawling): Scrape publication metadata, full-text LaTeX sources for all versions, and citation data.
2. Milestone 2 (Parsing and Standardization): Parse LaTeX sources into document element trees and standardize citations into BibTeX.
3. Milestone 3 (Reference Matching): Build and evaluate a reference matching model scored by MRR@5.

The corpus collected in this milestone provides the raw data for the parsing and modeling stages in Milestones 2 and 3. The pipeline collects native LaTeX source archives, complete version histories, and citation records from the Semantic Scholar Graph API instead of scraping compiled PDFs.

## Objectives and deliverables

For every assigned paper ID, the pipeline runs four steps:
- Query submission metadata and resolve the version list (v1 through the latest revision).
- Download source archives from the arXiv export endpoint, unpack them, and keep only `.tex` and `.bib` files. Figures, images, and compiled binaries are dropped to save disk space without altering the LaTeX content.
- Query Semantic Scholar to retrieve citation lists, paper titles, publication years, and external arXiv IDs.
- Deduplicate and export the collected records into two tables: `papers` and `references`.

The deliverables for Milestone 1 consist of:
- The Python crawler codebase in the group repository.
- This documentation site deployed on GitHub Pages with executable analysis notebooks.
- The exported `papers` and `references` dataset tables.

## Workload allocation and assigned ID ranges

The four team members each collected 5,000 papers, covering 20,000 papers in total.

| Full Name | Student ID (MSSV) | Primary Modules | Assigned Month | Start ID | End ID | Volume | Range Type |
|:---|:---|:---|:---|:---|:---|:---|:---|
| Nguyễn Đức Anh Khôi | 24120076 | `utils.py`, `downloader.py`, `performance.ipynb` | 2025-06 | 5788 | 10787 | 5,000 | Single month |
| Nguyễn Quang Phát | 24120117 | `crawler.py`, `implementation.md`, Project Architecture | 2025-05 | 10666 | 15665 | 5,000 | Single month |
| Đoàn Ngọc Bảo | 24120168 | `discovery.py`, `references.py`, `aggregate.py` | 2025-03 into 2025-04 | 21914 | 2522 | 5,000 | Cross-month boundary |
| Trần Nguyễn Quốc Khánh | 24120192 | `extractor.py`, `statistics.ipynb`, CI/CD Automation | 2025-03 | 1914 | 6913 | 5,000 | Single month |

## Dataset specifications

The pipeline generates two output tables:

### The `papers` table
Stores submission metadata and revision timestamps for each resolved paper:

| Field | Type | Constraint | Description |
|:---|:---|:---|:---|
| `arxiv_id` | String | Non-null, Primary Key | Base identifier (e.g. `2505.10666`) |
| `title` | String | Text | Title of the publication |
| `authors` | List of Strings | Array | Author names |
| `submission_date` | String | ISO 8601 Timestamp | Submission timestamp of the first version (v1) |
| `revised_dates` | List of Strings | Array | Timestamps of subsequent revisions |
| `venue` | String | Nullable | Journal or conference reference (`journal_ref`) |

### The `references` table
Stores citation records where the cited publication has a known arXiv ID (`ref_arxiv_id` is not null):

| Field | Type | Constraint | Description |
|:---|:---|:---|:---|
| `arxiv_id` | String | Non-null, Foreign Key | Identifier of the citing paper |
| `ref_arxiv_id` | String | Non-null | arXiv identifier of the cited paper |
| `ref_title` | String | Text | Title of the cited paper |
| `ref_authors` | List of Strings | Array | Authors of the cited paper |
| `ref_submission_date` | String | Text / Year | Publication year or date of the cited paper |
| `ref_s2_id` | String | Text | Semantic Scholar identifier (`paperId`) |

## Report structure

The report contains three main sections:
1. [System Implementation](implementation.md): Pipeline architecture, rate limiting, and handling of edge cases like PDF-only papers and archive extraction security.
2. [Dataset Statistics](statistics.ipynb): Summary tables and charts covering versions, authors, storage savings, and citation coverage.
3. [Performance Benchmark](performance.ipynb): Wall time, memory usage, and storage measurements on Google Colab CPU.
