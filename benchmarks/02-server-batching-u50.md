# 02 - Continuous batching under load (u50)

Host `Darwin-arm64` · `--parallel 4` · 28 samples over
60s at 2.0s intervals · raw CSV: `02-server-metrics-u50.csv`

| Gauge | Peak observed |
|:--|--:|
| `n_busy_slots_per_decode` (avg/decode) | 3.93 of 4 slots (98%) |
| `requests_processing` | 4 |
| `requests_deferred` | 46 |
| `kv_cache_usage_ratio` | n/a — not exported by llama.cpp `b10488` |
| `tokens_predicted_total` (final) | 9759 |

Highest sampled value was **3.93 of 4** slots. Note this gauge is llama.cpp's *average* busy slots per decode step, so the number below is the highest average we sampled, not an instantaneous maximum batch width. A peak near 1 means
requests were served one at a time -- either the load was too light to overlap, or
they arrived too far apart. A peak approaching `--parallel` means the scheduler was
genuinely packing concurrent requests into shared decode steps.
`requests_deferred` went above zero: more requests arrived than there were slots, so some waited. That wait is the queue time in your P95.

## Observation

The peak average decode width was 3.93/4 slots, direct evidence that continuous batching
kept every slot busy. Little's Law gives 34.8 effective concurrent requests, much larger
than four because it includes queued work; the peak of 46 deferred requests confirms that
interpretation. I trust the server gauge for active slot utilization and Little's Law for
total in-flight occupancy: they measure different parts of the same saturated system.
