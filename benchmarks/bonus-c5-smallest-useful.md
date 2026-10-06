# Bonus C5 - Quantization nhỏ nhất vẫn hữu ích

Host `Darwin-arm64` · Qwen3.5 0.8B · llama.cpp `b10488` · cùng prompt và cấu hình serving.

| Quantization | Kích thước | Decode | Câu trả lời Goodput@SLO |
|:--|--:|--:|:--|
| Q4_K_M | 0.50 GB | 62.1 tok/s | Liên hệ đúng goodput với request đạt mục tiêu SLO |
| UD-Q2_K_XL | 0.39 GB | 67.9 tok/s | Mô tả sai thành định dạng output dạng danh sách |

Q2 nhỏ hơn 22% và nhanh hơn 1.09x, nhưng prompt này cho thấy một lỗi ngữ nghĩa cụ thể ở
mức precision thấp hơn. Vì vậy Q4 là quantization nhỏ nhất tôi sẽ triển khai từ thí nghiệm
này. Đây là quality gate nhỏ có mục tiêu, không phải đánh giá model tổng quát; lựa chọn cho
production cần lặp lại trên bộ dữ liệu chuyên ngành lớn hơn.

Kết quả cho thấy tốc độ và kích thước file không đủ để định nghĩa “hữu ích”. Quantization
mạnh làm thay đổi weight đủ để model 0.8B mất một quan hệ cốt lõi dù câu chữ vẫn trôi chảy.
Tôi sẽ giữ Q4 và tìm lợi ích latency từ batching, độ dài prompt và Metal offload trước khi
đánh đổi độ chính xác đã quan sát được.
