# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · host `Darwin-arm64` · llama.cpp `b10488`
Settings: `threads=8` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 3139 | 108 / 235 | 30.8 / 37.2 | 1375 / 2467 / 2467 | 32.5 |
| UD-Q2_K_XL | 0.39 | 2038 | 79 / 90 | 16.6 / 19.1 | 1131 / 1289 / 1289 | 60.4 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.86x faster** than `Q4_K_M` here, for 0.11 GB less on disk.

## Observation

Q2 reduced size by 22%, improved median decode throughput 1.86x, and lowered TTFT P50
from 108 ms to 79 ms. It is not worth deploying for this model: on the same Goodput@SLO
prompt, Q4 connected goodput to SLO compliance, while Q2 incorrectly described it as a
list-oriented output format. I would retain Q4 because the quality loss outweighs 0.11 GB.
