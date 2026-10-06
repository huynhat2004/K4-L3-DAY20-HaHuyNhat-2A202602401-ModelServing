# Bonus B5/C8 - Semantic cache regime (offline)

The supplied offline demo used bag-of-words vectors and a simulated 250 ms inference
cost. At threshold 0.80 it produced 3/8 hits (38%), skipped three model calls, and saved
about 750 ms of simulated decode. The threshold sweep from 0.70 to 0.95 stayed flat at
3/8 because similarities from this stub were effectively 0 or 1.

This run validates the serving control flow, not semantic quality. A cache hit bypasses
prefill and decode entirely, unlike prefix/KV caching, but the stub cannot expose realistic
false-hit and false-miss boundaries. A production experiment needs a sentence encoder
such as BGE-M3 or Qwen3-Embedding, tenant-salted keys, and a labeled paraphrase/non-match
set. Shared unsalted caches can also leak cross-tenant information through returned content
or timing. The flat curve is evidence of the stub's limitation, not evidence that threshold
selection is unimportant.
