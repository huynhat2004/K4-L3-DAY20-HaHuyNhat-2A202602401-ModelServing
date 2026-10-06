# Bonus B1 - Prebuilt vs source build

Host `Darwin-arm64` · CPU `Apple M1`
Vector extensions detected: NEON
llama.cpp `b10488` both sides · `threads=8` ·
**both pinned to `ngl=0`** so this isolates the compiler ·
metric `tg128`, 3 repetitions

| Binary | Built for | tg128 (tok/s) | Relative |
|:--|--:|--:|--:|
| prebuilt release | runtime CPU dispatch | 26.4 | 1.00x |
| your source build | this CPU (`-DGGML_NATIVE=ON`) | 23.8 | 0.90x |

On this machine, the prebuilt binary is **1.11x faster**.

before: 26.4 tok/s (prebuilt release)
after:  23.8 tok/s (source build, -DGGML_NATIVE=ON)
speedup: 0.90x

Same source revision, same model, same backend, same `-ngl` -- the only difference
is what the compiler was allowed to assume about the CPU.


### Separately: what GPU offload is worth on the same binary

`tg128` on the source build at `-ngl 99` instead of `-ngl 0`:

| Source build | tg128 (tok/s) | vs its own CPU run |
|:--|--:|--:|
| `-ngl 0` (CPU) | 23.8 | 1.00x |
| `-ngl 99` (offloaded to MTL0: Apple M1 (10922 MiB, 10922 MiB free)) | 56.9 | 2.39x |

This number is **not** part of the B1 comparison above -- it is a different knob.
Reporting it separately is the point: a compiler flag and an accelerator are not
interchangeable explanations for a speedup.


## Explanation

The native build did not win: it was 10% slower despite enabling M1-specific NEON,
dot-product and FP16 vector arithmetic. The prebuilt arm64 release already uses an
optimized runtime dispatch path and Accelerate, so `-mcpu=native` adds little to a
memory-heavy token-generation workload; compiler/code-layout differences and normal
thermal variation can outweigh that small instruction-level opportunity. The separate
Metal result is much larger (2.39x) because offload changes the execution resource and
memory-parallelism regime, not merely compiler assumptions. This is why the GPU number
must not be presented as the B1 compiler speedup.
