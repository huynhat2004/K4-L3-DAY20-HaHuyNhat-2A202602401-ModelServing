# Bonus C5 - Smallest useful quantization

Host `Darwin-arm64` · Qwen3.5 0.8B · llama.cpp `b10488` · identical prompt and serving
settings.

| Quantization | Size | Decode | Goodput@SLO answer |
|:--|--:|--:|:--|
| Q4_K_M | 0.50 GB | 32.5 tok/s | Correctly tied goodput to requests meeting SLO targets |
| UD-Q2_K_XL | 0.39 GB | 60.4 tok/s | Incorrectly called it a list-oriented output format |

Q2 is 22% smaller and 1.86x faster, but this prompt exposes a concrete semantic failure
at the next lower tested precision. Q4 is therefore the smallest quantization I would
deploy from this experiment. This is a small targeted quality gate, not a general model
evaluation; production selection would repeat it over a larger domain-specific set.

The result illustrates why speed and file size alone cannot define “useful.” Aggressive
quantization perturbs weights enough that a small 0.8B model can lose a core relationship
even while producing fluent text. I would keep Q4 and seek latency gains from batching,
prompt length, and Metal offload before sacrificing this observed correctness.
