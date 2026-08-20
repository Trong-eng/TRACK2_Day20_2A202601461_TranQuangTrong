# 03 - Integrate: RAG pipeline run

Host `Darwin-arm64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 1922.3 | 1922.4 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.0 | 849.5 | 849.6 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.0 | 850.2 | 850.3 |

Mean per stage (ms): embed **0.0** · retrieve **0.0** ·
llm **1207.3** · total **1207.4**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Goodput@SLO counts only the requests per second that met the TTFT and TPOT targets. Throughput at saturation ignores SLOs.

**What problem does PagedAttention actually solve?**

> PagedAttention stores the KV cache in non-contiguous pages, which removes the internal fragmentation that wasted most GPU memory.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps because prefill is compute-bound and decode is memory-bandwidth-bound.


## Which N16-N19 pieces are real

N16 Cloud/IaC, N17 Data pipelines, N18 Lakehouse và N19 Vector + features đều là
stub trong lần chạy này: localhost-only, `TOY_DOCS` in-memory, toy dict và keyword
overlap retrieval. N20 `llama-server` là real. LLM chiếm 100% latency (1207,3 ms trung
bình), đúng với kỳ vọng; để giảm latency 2x, tôi sẽ tối ưu stage LLM trước bằng model
nhỏ hơn/quantization hoặc giảm output/context tokens.
