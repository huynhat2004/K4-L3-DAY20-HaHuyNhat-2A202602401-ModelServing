# 03 - Tích hợp: chạy pipeline RAG

Host `Darwin-arm64` · llama.cpp `b10488` ·
retrieval backend: **khớp từ khóa** · 3 truy vấn

| Truy vấn | Context truy xuất | embed (ms) | retrieve (ms) | llm (ms) | tổng (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 2587.1 | 2587.2 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.0 | 1797.4 | 1797.5 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.1 | 2003.0 | 2003.1 |

Trung bình mỗi giai đoạn (ms): embed **0.0** · retrieve **0.0** ·
llm **2129.2** · tổng **2129.3**
Giai đoạn chi phối: **llm** (100% tổng thời gian)

## Câu trả lời nhận được

**Why is goodput more useful than raw throughput?**

> Based on the provided context, **Goodput** is more useful than raw throughput because it focuses on **SLOs (Service Level Objectives)** rather than ignoring them.

Specifically, the text states:
> "Goodput@SLO counts only the requests per second that met the TTFT and TPOT targets. Throughput at saturation ignores SLOs."

This implies that while raw throughput measures total data volume (which can 

**What problem does PagedAttention actually solve?**

> PagedAttention solves the problem of **internal fragmentation in GPU memory** caused by storing the key-value cache (KV cache) in non-contiguous pages.

By using non-contiguous pages, the model avoids the wasted space that would exist if the KV cache were stored in a contiguous block of memory. This optimization allows the engine to utilize more of the available GPU memory, which is particularly b

**When does splitting prefill and decode help?**

> Based on the provided context, splitting prefill and decode helps when **prefill is compute-bound and decode is memory-bandwidth-bound**.

This is because the context explicitly states that "Disaggregated serving splits prefill and decode onto separate pools because prefill is compute-bound and decode is memory-bandwidth-bound." By having them run in separate pools, the system avoids the overhead 


## Thành phần N16-N19 nào là thật

- N16 Cloud/IaC: mô phỏng (chỉ có tiến trình local).
- N17 Data pipeline: mô phỏng (tài liệu mẫu trong bộ nhớ).
- N18 Lakehouse: mô phỏng (không có bảng hoặc lakehouse bên ngoài).
- N19 Vector + features: mô phỏng (khớp từ khóa, không có embedding service).
- N20 Serving: thật (`llama-server` qua `/v1/chat/completions`).

LLM chiếm 2129.2 ms, gần 100% latency trung bình, đúng kỳ vọng vì retrieval mô phỏng chạy
trong bộ nhớ. Để giảm một nửa latency end-to-end, tôi sẽ giới hạn độ dài output, giữ prefix
prompt dùng chung để tái sử dụng cache và chỉ thử quantization nhỏ hơn sau quality gate.
Tối ưu retrieval không thể tiết kiệm đáng kể trong pipeline đã đo.
