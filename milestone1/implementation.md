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

<!-- Paper withdrawn, PDF-only, single-file gzip, ID không tồn tại, cross-month range, ... -->
