# 02 - Serve: load test + saturation reading

Host `Darwin-arm64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=8` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 71 | 1.29 | 6300 | 10000 | 13000 | 8.5 | 0.0% |
| 50 | 83 | 1.42 | 29000 | 36000 | 40000 | 34.8 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.10x** (22% of linear) |
| P95 latency | **3.60x** |
| Effective concurrency at 50 users | 34.8 vs `--parallel 4` slots (occupancy/slot ratio 8.70) |

**Saturated.** Throughput delivered only 1.10x for 5x the offered load, and effective concurrency (34.8) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 1.10x while P95 moved 3.60x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Reading

The server is already near saturation at 10 users and is clearly saturated by 50:
offered load rose 5x, but RPS rose only 1.10x while P95 grew 3.60x to 36 s. At 50 users,
34.8 requests were effectively in flight against four decode slots; metrics showed
3.93/4 busy slots and 46 deferred requests, so most added latency is queueing. For a
10 s P95 SLO, I would first increase `--parallel` while memory permits, then remeasure;
it directly adds schedulable slots, whereas more CPU threads cannot relieve Metal-bound
decode and were slower in the thread sweep.
