# Bonus - Khảo sát độ dài context (chi phí prefill)

Host `Darwin-arm64` · llama.cpp `b10488` ·
`threads=8` `ngl=99` · RAM 16.0 GB

| Token trong prompt | Prefill (tok/s) | Phần TTFT (ms) | So với tăng tuyến tính |
|:--|--:|--:|--:|
| 256 | 1092.1 | 234.4 | 1.00x |
| 1024 | 1122.4 | 912.3 | 0.97x |
| 2048 | 1102.9 | 1857.0 | 0.99x |
| 4096 | 811.3 | 5048.6 | 1.35x |
| 8192 | 972.4 | 8424.3 | 1.12x |

Tại 8192 token, prefill tốn **8424 ms**, bằng 1.12x dự đoán tuyến tính từ điểm nhỏ nhất.
Phần vượt thêm cho thấy thành phần O(N^2) của attention bắt đầu xuất hiện, và toàn bộ thời
gian này nằm trong TTFT trước khi người dùng thấy token đầu tiên.

Đây là con số cần nhớ khi muốn đưa thêm context truy xuất vào prompt RAG chỉ vì context
window cho phép. Mỗi request phải trả toàn bộ chi phí prefill trước khi token đầu xuất hiện.

## Kết luận

Prefill gần tuyến tính tới 2048 token (1857 ms), sau đó bẻ cong rõ tại 4096: 5049 ms bằng
1.35x dự đoán tuyến tính và đã vượt tổng latency trung bình 2129 ms của RAG context ngắn.
Tại 8192 token, người dùng chờ 8424 ms trước decode. Tôi sẽ giới hạn context truy xuất gần
2048 token, xếp hạng chunk trước khi tạo prompt và chỉ chấp nhận TTFT tăng 2.7x từ 2048 lên
4096 token khi đo được mức tăng relevance tương xứng.
