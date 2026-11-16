# Benchmark arXiv trên Colab

**15,950 / 20,000 ID đã đo** · 15,791 ID đã chốt · 638 lượt lô đã đồng bộ.

| Nguồn ID | Đã đo | Đã chốt | SUCCESS | PDF_ONLY | FAILED | INCOMPLETE |
|---|---:|---:|---:|---:|---:|---:|
| arxiv_2503_2504_summary.csv | 5000 | 4987 | 4510 | 477 | 4 | 9 |
| arxiv_2503_summary.csv | 5000 | 4973 | 4500 | 473 | 10 | 17 |
| arxiv_2505_summary.csv | 5000 | 4885 | 4512 | 373 | 30 | 85 |
| arxiv_2506_summary.csv | 950 | 946 | 872 | 74 | 4 | 0 |

- Tổng thời gian profiler của các lượt hợp lệ (gồm lượt thử lại) · 131,130.5 s
- Tổng thời gian máy quan sát được, kể cả lượt dở và lưu Drive/Git · 141,839.3 s
- Khoảng thời gian lịch từ lúc bắt đầu đến mẫu cuối (gồm lúc nhóm nghỉ) · 233,628.8 s
- Kết quả theo ID, danh tính phiên và biên nhận Drive nằm trong `blocks/` và `attempts/`.
- Số liệu cập nhật sau mỗi lần chốt lô. Heartbeat của lượt đang chạy nằm trong thư mục attempts.
- FAILED/INCOMPLETE được giữ nguyên sau khi dùng hết số lượt thử. PDF_ONLY theo định nghĩa profiler của commit này.

Mở `progress.json` để đọc thời gian các pha, RAM, dung lượng, số lần gọi mạng và thời gian chờ.
