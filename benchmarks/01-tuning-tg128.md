# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` · host `Darwin-arm64` · llama.cpp `b10488`
CPU: **8 physical · 8 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 58.4 | 100% |
| 4 | 57.2 | 98% |
| 8 | 55.5 | 95% |
| 16 | 54.2 | 93% |

**Best**: `-t 1` at 58.4 tok/s
**Slowest tested**: `-t 16` at 54.2 tok/s (1.08x spread)
**Against the physical-core default** (`-t 8`, 55.5 tok/s): 1.05x

Use this in your run:

```bash
LAB_N_THREADS=1 make bench
```

## Explanation

The curve peaks at one thread, not at the eight physical cores: 58.4 tok/s at `-t 1`
versus 55.5 at `-t 8` and 54.2 at `-t 16`. This run uses `ngl=99`, so Metal executes
nearly all model layers. CPU thread count is therefore not widening the dominant decode
work. Extra host threads add scheduling and synchronization overhead while competing for
the same unified-memory subsystem. The small 1.08x total spread also shows that GPU and
memory bandwidth, rather than CPU parallelism, dominate this workload.
