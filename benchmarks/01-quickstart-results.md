# 01 - Measure: latency baseline

Model `Gemma 4 E2B` · host `Darwin-arm64` · llama.cpp `b10488`
Settings: `threads=8` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `UD-Q4_K_XL` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 4157 | 144 / 295 | 23.3 / 25.0 | 1604 / 1855 / 1855 | 43.0 |
| UD-Q2_K_XL | 2.24 | 3085 | 146 / 366 | 19.7 / 19.9 | 1384 / 1620 / 1620 | 50.7 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.18x faster** than `UD-Q4_K_XL` here, for 0.73 GB less on disk.

## Your observation

Trên Apple M3, UD-Q2_K_XL nhỏ hơn 0,73 GB (24,6%) và decode nhanh hơn 1,18x so với
UD-Q4_K_XL (50,7 so với 43,0 tok/s); load model cũng nhanh hơn khoảng 1,35x.
TTFT P50 gần như tương đương, còn TPOT tốt hơn rõ rệt. Với cùng câu hỏi về continuous
batching, cả hai trả lời đúng; Q4 diễn đạt cụ thể hơn, Q2 khái quát hơn. Vì vậy Q2
đáng dùng khi ưu tiên tốc độ/dung lượng và chấp nhận giảm độ chi tiết nhẹ.
