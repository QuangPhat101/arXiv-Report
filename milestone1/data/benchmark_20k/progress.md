# Benchmark arXiv trên Colab

**3,225 / 20,000 ID đã đo** · 3,202 ID đã chốt · 129 lượt lô đã đồng bộ.

| Nguồn ID | Đã đo | Đã chốt | SUCCESS | PDF_ONLY | FAILED | INCOMPLETE |
|---|---:|---:|---:|---:|---:|---:|
| arxiv_2503_2504_summary.csv | 0 | 0 | 0 | 0 | 0 | 0 |
| arxiv_2503_summary.csv | 3225 | 3202 | 2909 | 293 | 6 | 17 |
| arxiv_2505_summary.csv | 0 | 0 | 0 | 0 | 0 | 0 |
| arxiv_2506_summary.csv | 0 | 0 | 0 | 0 | 0 | 0 |

- Tổng thời gian profiler của các lượt hợp lệ (gồm lượt thử lại) · 28,538.9 s
- Tổng thời gian máy quan sát được, kể cả lượt dở và lưu Drive/Git · 35,915.6 s
- Khoảng thời gian lịch từ lúc bắt đầu đến mẫu cuối (gồm lúc nhóm nghỉ) · 83,836.8 s
- Kết quả theo ID, danh tính phiên và biên nhận Drive nằm trong `blocks/` và `attempts/`.
- Số liệu cập nhật sau mỗi lần chốt lô. Heartbeat của lượt đang chạy nằm trong thư mục attempts.
- FAILED/INCOMPLETE được giữ nguyên sau khi dùng hết số lượt thử. PDF_ONLY theo định nghĩa profiler của commit này.

Mở `progress.json` để đọc thời gian các pha, RAM, dung lượng, số lần gọi mạng và thời gian chờ.
