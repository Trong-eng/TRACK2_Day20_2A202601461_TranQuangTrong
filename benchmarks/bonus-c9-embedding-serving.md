# Bonus C9 - Embedding serving regime

Host `Darwin-arm64` · Apple Metal · llama.cpp `b10488`

The embedding server reused the Gemma 4 E2B GGUF in pooling mode and returned
1536-dimensional vectors. This is a prefill-bound workload: one forward pass per
text, with no decode loop and no chat-style KV-cache generation.

| Batch | Latency (ms) | Throughput (texts/s) |
|--:|--:|--:|
| 1 | 94.8 | 10.5 |
| 2 | 99.0 | 20.2 |
| 4 | 153.8 | 26.0 |
| 8 | 360.2 | 22.2 |
| 16 | 750.9 | 21.3 |

## Finding

Static batching improved throughput from 10.5 texts/s at batch 1 to a peak of 26.0
texts/s at batch 4 (2.48x). Larger batches reduced throughput on this laptop because
the added prefill work and memory pressure outweighed batching efficiency. This differs
from chat serving, where decode cost and continuous slot scheduling dominate. The demo
uses a chat model as an embedding backend, so a dedicated embedding model would be the
next quality/efficiency improvement for production retrieval.
