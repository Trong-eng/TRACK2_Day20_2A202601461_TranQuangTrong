# Bonus - GPU offload sweep

Host `Darwin-arm64` · backend(s) `apple_metal` ·
llama.cpp `b10488` · `threads=8` · metric `tg128`

| -ngl | tg128 (tok/s) | vs -ngl 0 | vs best |
|:--|--:|--:|--:|
| 0 | 34.9 | 1.00x | 80% |
| 8 | 27.9 | 0.80x | 64% |
| 16 | 36.6 | 1.05x | 84% |
| 24 | 40.8 | 1.17x | 94% |
| 32 | 39.0 | 1.12x | 90% |
| 99 | 43.4 | 1.24x | 100% |

Best: `-ngl 99` at 43.4 tok/s
-- 1.24x faster than CPU-only.

Where the curve flattens tells you the model ran out of layers to move. Where it
*peaks below* full offload tells you something did not fit and the accelerator
started paying to fetch weights it could not hold.

## Your finding

Full offload (`-ngl 99`) là tốt nhất trên máy này: 43,4 tok/s, nhanh hơn 1,24x so với
CPU-only `-ngl 0` ở 34,9 tok/s. Các mức partial không tăng đều; `-ngl 8` còn chậm
hơn CPU-only, sau đó tốc độ tăng khi chuyển thêm nhiều layer sang Metal. Đây không
phải do thiếu VRAM vì Apple M3 còn đủ bộ nhớ cho model; nguyên nhân hợp lý hơn là
chi phí phối hợp và di chuyển dữ liệu giữa CPU và GPU khi offload chưa đầy đủ.
