# Benchmark arXiv trên Colab

**15,425 / 20,000 ID đã đo** · 15,267 ID đã chốt · 617 lượt lô đã đồng bộ.

| Nguồn ID | Đã đo | Đã chốt | SUCCESS | PDF_ONLY | FAILED | INCOMPLETE |
|---|---:|---:|---:|---:|---:|---:|
| arxiv_2503_2504_summary.csv | 5000 | 4987 | 4510 | 477 | 4 | 9 |
| arxiv_2503_summary.csv | 5000 | 4973 | 4500 | 473 | 10 | 17 |
| arxiv_2505_summary.csv | 5000 | 4885 | 4512 | 373 | 30 | 85 |
| arxiv_2506_summary.csv | 425 | 422 | 401 | 21 | 3 | 0 |

- Tổng thời gian profiler của các lượt hợp lệ (gồm lượt thử lại) · 127,415.9 s
- Tổng thời gian máy quan sát được, kể cả lượt dở và lưu Drive/Git · 137,820.3 s
- Khoảng thời gian lịch từ lúc bắt đầu đến mẫu cuối (gồm lúc nhóm nghỉ) · 186,396.4 s
- Kết quả theo ID, danh tính phiên và biên nhận Drive nằm trong `blocks/` và `attempts/`.
- Số liệu cập nhật sau mỗi lần chốt lô. Heartbeat của lượt đang chạy nằm trong thư mục attempts.
- FAILED/INCOMPLETE được giữ nguyên sau khi dùng hết số lượt thử. PDF_ONLY theo định nghĩa profiler của commit này.

Mở `progress.json` để đọc thời gian các pha, RAM, dung lượng, số lần gọi mạng và thời gian chờ.
