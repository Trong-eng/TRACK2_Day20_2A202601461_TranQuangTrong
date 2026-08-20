# Bonus B1 - Prebuilt vs source build

Host `Darwin-arm64` · CPU `Apple M3`
Vector extensions detected: NEON
llama.cpp `b10488` both sides · `threads=8` ·
**both pinned to `ngl=0`** so this isolates the compiler ·
metric `tg128`, 3 repetitions

| Binary | Built for | tg128 (tok/s) | Relative |
|:--|--:|--:|--:|
| prebuilt release | runtime CPU dispatch | 21.0 | 1.00x |
| your source build | this CPU (`-DGGML_NATIVE=ON`) | 24.9 | 1.19x |

On this machine, the source build is **1.19x faster**.

before: 21.0 tok/s (prebuilt release)
after:  24.9 tok/s (source build, -DGGML_NATIVE=ON)
speedup: 1.19x

Same source revision, same model, same backend, same `-ngl` -- the only difference
is what the compiler was allowed to assume about the CPU.


### Separately: what GPU offload is worth on the same binary

`tg128` on the source build at `-ngl 99` instead of `-ngl 0`:

| Source build | tg128 (tok/s) | vs its own CPU run |
|:--|--:|--:|
| `-ngl 0` (CPU) | 24.9 | 1.00x |
| `-ngl 99` (offloaded to MTL0: Apple M3 (12124 MiB, 12123 MiB free)) | 35.8 | 1.43x |

This number is **not** part of the B1 comparison above -- it is a different knob.
Reporting it separately is the point: a compiler flag and an accelerator are not
interchangeable explanations for a speedup.


## Your explanation

Apple M3 có NEON, và source build với `-DGGML_NATIVE=ON` cho compiler biết rõ hơn các
đặc tính CPU thay vì dùng runtime CPU dispatch tổng quát như prebuilt. Vì vậy source
build đạt 24,9 tok/s so với 21,0 tok/s, tức 1,19x. Mức tăng vừa phải cho thấy decode
vẫn chịu ảnh hưởng đáng kể bởi memory bandwidth; tối ưu instruction/vectorization
chỉ cải thiện phần overhead tính toán. GPU offload là một knob khác, đạt 35,8 tok/s
(1,43x so với source CPU), nên không được dùng để giải thích speedup compiler.
