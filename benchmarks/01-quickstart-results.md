# 01 - Đo latency cơ sở

Model `Qwen3.5 0.8B` · host `Darwin-arm64` · llama.cpp `b10488`
Thiết lập: `threads=8` `ngl=99` `ctx=2048`
`max_tokens=64` · đã loại lượt khởi động
Request hoàn tất: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Kích thước (GB) | Tải model (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 2076 | 82 / 91 | 16.1 / 18.1 | 1050 / 1226 / 1226 | 62.1 |
| UD-Q2_K_XL | 0.39 | 1017 | 78 / 110 | 14.7 / 17.8 | 1002 / 1218 / 1218 | 67.9 |

- **TTFT** = prefill. Prompt ngắn giữ giá trị này thấp; RAG context dài làm nó tăng mạnh.
- **TPOT** = chi phí decode cho mỗi output token, bị giới hạn bởi memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decode **nhanh hơn 1.09x** so với `Q4_K_M`, đồng thời nhỏ hơn 0.11 GB.

## Nhận xét

Q2 giảm 22% kích thước nhưng throughput decode trung vị chỉ tăng 1.09x; TTFT P95 còn
xấu hơn (110 ms so với 91 ms). Q2 không đáng triển khai trong trường hợp này: với cùng
prompt Goodput@SLO, Q4 liên hệ đúng goodput với việc đáp ứng SLO, còn Q2 mô tả sai thành
định dạng output dạng danh sách. Tôi chọn Q4 vì suy giảm chất lượng lớn hơn lợi ích 0.11 GB.
