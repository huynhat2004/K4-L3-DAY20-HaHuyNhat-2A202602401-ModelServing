# 01 - Tinh chỉnh: khảo sát số thread

Model `Qwen3.5-0.8B-Q4_K_M.gguf` · host `Darwin-arm64` · llama.cpp `b10488`
CPU: **8 physical · 8 logical** core · `ngl=99` · metric `tg128`

| Số thread (-t) | tg128 (tok/s) | So với tốt nhất |
|:--|--:|--:|
| 1 | 58.4 | 100% |
| 4 | 57.2 | 98% |
| 8 | 55.5 | 95% |
| 16 | 54.2 | 93% |

**Tốt nhất**: `-t 1` đạt 58.4 tok/s
**Chậm nhất đã thử**: `-t 16` đạt 54.2 tok/s (chênh lệch 1.08x)
**So với mặc định theo số core vật lý** (`-t 8`, 55.5 tok/s): 1.05x

Dùng cấu hình sau:

```bash
LAB_N_THREADS=1 make bench
```

## Giải thích

Đường cong đạt đỉnh ở một thread thay vì tám core vật lý: 58.4 tok/s tại `-t 1`, so với
55.5 tại `-t 8` và 54.2 tại `-t 16`. Phép đo dùng `ngl=99`, nên Metal thực thi gần như
toàn bộ layer. Tăng số CPU thread không mở rộng phần decode chi phối; thread host bổ sung
chỉ tăng chi phí lập lịch, đồng bộ và cạnh tranh unified memory. Chênh lệch tổng 1.08x
cũng cho thấy GPU và memory bandwidth chi phối workload này hơn CPU parallelism.
