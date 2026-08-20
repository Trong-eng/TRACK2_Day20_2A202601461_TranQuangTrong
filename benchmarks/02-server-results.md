# 02 - Serve: load test + saturation reading

Host `Darwin-arm64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=8` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 59 | 1.00 | 8900 | 12000 | 13000 | 8.7 | 0.0% |
| 50 | 61 | 1.04 | 29000 | 48000 | 50000 | 29.3 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.05x** (21% of linear) |
| P95 latency | **4.00x** |
| Effective concurrency at 50 users | 29.3 vs `--parallel 4` slots (occupancy/slot ratio 7.33) |

**Saturated.** Throughput delivered only 1.05x for 5x the offered load, and effective concurrency (29.3) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 1.05x while P95 moved 4.00x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Your reading

Server đã bão hòa ở mức tải 50 users (và có thể bắt đầu từ dưới mức đó): tải tăng 5x
nhưng throughput chỉ tăng 1,05x, trong khi P95 tăng 4,00x từ 12.000 lên 48.000 ms.
Effective concurrency là 29,3 so với 4 slots và metrics ghi nhận peak 3,99/4 slots
cùng 46 request deferred. Vì vậy latency tăng chủ yếu là queue time. Để tăng goodput
trong SLO, tôi sẽ thử tăng `--parallel` trước và kiểm tra lại RAM/queue; đổi thread
không giải quyết hàng đợi do slot đã bận.
