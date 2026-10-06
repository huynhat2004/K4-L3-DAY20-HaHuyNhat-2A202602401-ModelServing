# 03 - Integrate: RAG pipeline run

Host `Darwin-arm64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 2587.1 | 2587.2 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.0 | 1797.4 | 1797.5 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.1 | 2003.0 | 2003.1 |

Mean per stage (ms): embed **0.0** · retrieve **0.0** ·
llm **2129.2** · total **2129.3**
Dominant stage: **llm** (100% of total)

## Answers returned

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


## Which N16-N19 pieces are real

- N16 Cloud/IaC: stub (local process only).
- N17 Data pipeline: stub (in-memory sample documents).
- N18 Lakehouse: stub (no external table or lakehouse).
- N19 Vector + features: stub (keyword overlap fallback, no embedding service).
- N20 Serving: real (`llama-server` through `/v1/chat/completions`).

The LLM accounts for 2129.2 ms, effectively 100% of mean latency, as expected because
the stub retrieval is in-memory. To halve end-to-end latency I would target generation:
cap output length, preserve shared prompt prefixes for cache reuse, and test a smaller
quantization only behind a quality gate. Optimizing retrieval cannot save meaningful time
in this measured pipeline.
