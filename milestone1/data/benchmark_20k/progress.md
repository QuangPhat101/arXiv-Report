# Benchmark arXiv trên Colab

**19,225 / 20,000 ID đã đo** · 19,042 ID đã chốt · 769 lượt lô đã đồng bộ.

| Nguồn ID | Đã đo | Đã chốt | SUCCESS | PDF_ONLY | FAILED | INCOMPLETE |
|---|---:|---:|---:|---:|---:|---:|
| arxiv_2503_2504_summary.csv | 5000 | 4987 | 4510 | 477 | 4 | 9 |
| arxiv_2503_summary.csv | 5000 | 4973 | 4500 | 473 | 10 | 17 |
| arxiv_2505_summary.csv | 5000 | 4885 | 4512 | 373 | 30 | 85 |
| arxiv_2506_summary.csv | 4225 | 4197 | 3892 | 305 | 19 | 9 |

- Tổng thời gian profiler của các lượt hợp lệ (gồm lượt thử lại) · 156,251.1 s
- Tổng thời gian máy quan sát được, kể cả lượt dở và lưu Drive/Git · 167,836.2 s
- Khoảng thời gian lịch từ lúc bắt đầu đến mẫu cuối (gồm lúc nhóm nghỉ) · 247,233.8 s
- Kết quả theo ID, danh tính phiên và biên nhận Drive nằm trong `blocks/` và `attempts/`.
- Số liệu cập nhật sau mỗi lần chốt lô. Heartbeat của lượt đang chạy nằm trong thư mục attempts.
- FAILED/INCOMPLETE được giữ nguyên sau khi dùng hết số lượt thử. PDF_ONLY theo định nghĩa profiler của commit này.

Mở `progress.json` để đọc thời gian các pha, RAM, dung lượng, số lần gọi mạng và thời gian chờ.
