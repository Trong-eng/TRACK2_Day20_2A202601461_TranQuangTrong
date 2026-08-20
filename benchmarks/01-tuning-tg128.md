# 01 - Tune: thread-count sweep

Model `gemma-4-E2B-it-UD-Q4_K_XL.gguf` · host `Darwin-arm64` · llama.cpp `b10488`
CPU: **8 physical · 8 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 45.0 | 99% |
| 4 | 45.3 | 100% |
| 8 | 42.8 | 94% |
| 16 | 37.8 | 83% |

**Best**: `-t 4` at 45.3 tok/s
**Slowest tested**: `-t 16` at 37.8 tok/s (1.20x spread)
**Against the physical-core default** (`-t 8`, 42.8 tok/s): 1.06x

Use this in your run:

```bash
LAB_N_THREADS=4 make bench
```

## Your explanation

Knee nằm ở `-t 4` với 45,3 tok/s. Đây là kết quả hơi bất ngờ vì máy có 8 physical
cores: `-t 8` giảm còn 42,8 tok/s và `-t 16` giảm mạnh còn 37,8 tok/s. Decode bị giới
hạn bởi memory bandwidth và chi phí scheduling/cache, nên thêm thread không tạo thêm
throughput; các thread bổ sung cạnh tranh tài nguyên và làm oversubscription. Giảm từ
8 xuống 4 cho speedup 1,06x (45,3/42,8), còn `-t 16` chậm hơn 17% so với mức tốt nhất.
