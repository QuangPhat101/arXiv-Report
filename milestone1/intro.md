# Milestone 1 — Data Crawling

## Giới thiệu

Milestone 1 tập trung vào việc cào dữ liệu từ arXiv — kho lưu trữ mở chứa hơn hai triệu bài báo khoa học. Mục tiêu là thu thập full-text LaTeX source, metadata, và reference metadata cho toàn bộ dải paper ID được giao.

## Mục tiêu

- Resolve các arXiv ID được giao, xác định tất cả version của mỗi paper.
- Tải source archive, chỉ giữ file `.tex` và `.bib` (bỏ figure).
- Cào reference metadata qua Semantic Scholar API.
- Tổng hợp thành 2 bảng: `papers` và `references`.

## Dải ID

| Thành viên | Tháng | Start ID | End ID |
|:-----------|:------|:---------|:-------|
| Member1 | 2025-06 | 5788 | 10787 |
| Member2 | 2025-05 | 10666 | 15665 |
| Member3 | 2025-03 → 2025-04 | 21914 | 2522 |
| Member4 | 2025-03 | 1914 | 6913 |
