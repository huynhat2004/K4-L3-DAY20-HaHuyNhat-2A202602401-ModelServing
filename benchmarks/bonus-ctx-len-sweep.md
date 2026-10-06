# Bonus - Context-length sweep (prefill cost)

Host `Darwin-arm64` · llama.cpp `b10488` ·
`threads=8` `ngl=99` · RAM 16.0 GB

| Prompt tokens | Prefill (tok/s) | TTFT contribution (ms) | vs linear scaling |
|:--|--:|--:|--:|
| 256 | 1092.1 | 234.4 | 1.00x |
| 1024 | 1122.4 | 912.3 | 0.97x |
| 2048 | 1102.9 | 1857.0 | 0.99x |
| 4096 | 811.3 | 5048.6 | 1.35x |
| 8192 | 972.4 | 8424.3 | 1.12x |

At 8192 tokens, prefill costs **8424 ms** --
1.12x what linear scaling from the smallest point would predict. That excess
is attention's O(N^2) term becoming visible, and every millisecond of it lands in TTFT
before the user sees a single token.

Either way, this is the number to remember when someone proposes stuffing more retrieved
context into a RAG prompt "because the context window allows it". Prefill is paid in full,
on every request, before the first token appears.

## Finding

Prefill stays close to linear through 2048 tokens (1857 ms), then bends sharply at 4096:
5049 ms is 1.35x the linear prediction and already exceeds the complete 2129 ms mean of
the short-context RAG run. At 8192 tokens, users wait 8424 ms before decode. I would cap
retrieved context near 2048 tokens, rank chunks before prompt construction, and require
measured relevance gains before accepting the 2.7x TTFT jump from 2048 to 4096 tokens.
