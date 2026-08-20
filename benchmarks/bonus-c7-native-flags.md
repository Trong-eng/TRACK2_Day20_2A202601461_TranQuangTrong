# Bonus C7 - Native instruction-set build comparison

Host `Darwin-arm64` · CPU `Apple M3` · NEON · llama.cpp `b10488`

Both builds used the same model, `threads=8`, `-ngl 0`, Release mode, and `tg128`.
The only build difference was `GGML_NATIVE`:

| Build | Flag | tg128 (tok/s) |
|:--|:--|--:|
| Native ON | `-DGGML_NATIVE=ON` | 30.58 ± 6.77 |
| Native OFF | `-DGGML_NATIVE=OFF` | 27.18 ± 13.90 |

The native build was approximately **1.13x faster** in this repeat (`30.58 / 27.18`).
Both binaries reported the same `MTL,BLAS` backend and were run with `-ngl 0`, so the
comparison kept the runtime flags and model constant. The large standard deviations,
especially for Native OFF, mean this is a directional result rather than a precise
production benchmark.

## Finding

`GGML_NATIVE=ON` was faster in the controlled repeat, consistent with the compiler
being allowed to target the Apple M3 NEON instruction set. However, the high variance
and Metal backend initialization indicate that system/runtime noise is material on this
machine. The safe conclusion is a modest native-build advantage, not a guaranteed 1.13x
speedup for every workload.
