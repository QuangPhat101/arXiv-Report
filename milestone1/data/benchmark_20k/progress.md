# Benchmark arXiv trên Colab

**20,000 / 20,000 ID đã đo** · 19,971 ID đã chốt · 852 lượt lô đã đồng bộ.

| Nguồn ID | Đã đo | Đã chốt | SUCCESS | PDF_ONLY | FAILED | INCOMPLETE |
|---|---:|---:|---:|---:|---:|---:|
| arxiv_2503_2504_summary.csv | 5000 | 5000 | 4523 | 477 | 0 | 0 |
| arxiv_2503_summary.csv | 5000 | 5000 | 4526 | 474 | 0 | 0 |
| arxiv_2505_summary.csv | 5000 | 5000 | 4619 | 380 | 0 | 1 |
| arxiv_2506_summary.csv | 5000 | 4971 | 4595 | 376 | 20 | 9 |

- Tổng thời gian profiler của các lượt hợp lệ (gồm lượt thử lại) · 163,913.1 s
- Tổng thời gian máy quan sát được, kể cả lượt dở và lưu Drive/Git · 175,967.7 s
- Khoảng thời gian lịch từ lúc bắt đầu đến mẫu cuối (gồm lúc nhóm nghỉ) · 251,695.9 s
- Kết quả theo ID, danh tính phiên và biên nhận Drive nằm trong `blocks/` và `attempts/`.
- Số liệu cập nhật sau mỗi lần chốt lô. Heartbeat của lượt đang chạy nằm trong thư mục attempts.
- FAILED/INCOMPLETE được giữ nguyên sau khi dùng hết số lượt thử. PDF_ONLY theo định nghĩa profiler của commit này.

Mở `progress.json` để đọc thời gian các pha, RAM, dung lượng, số lần gọi mạng và thời gian chờ.
