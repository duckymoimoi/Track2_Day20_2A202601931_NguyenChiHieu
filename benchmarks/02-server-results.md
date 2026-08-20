# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=8` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 103 | 1.75 | 4400 | 6800 | 7200 | 8.1 | 0.0% |
| 50 | 108 | 1.85 | 25000 | 27000 | 29000 | 38.6 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.05x** (21% of linear) |
| P95 latency | **3.97x** |
| Effective concurrency at 50 users | 38.6 vs `--parallel 4` slots (occupancy/slot ratio 9.64) |

**Saturated.** Throughput delivered only 1.05x for 5x the offered load, and effective concurrency (38.6) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 1.05x while P95 moved 3.97x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## My reading

Server đã bão hòa trước hoặc tại 50 users: offered load tăng 5x nhưng RPS chỉ tăng 1.05x,
trong khi P95 tăng 3.97x lên 27 s. Peak 3.93/4 slot và 46 request deferred cho thấy phần
latency tăng thêm chủ yếu là queue time. Với SLO P95 dưới 7 s, run 10 users còn đạt nhưng
run 50 users không đạt. Tôi sẽ thử tăng `--parallel` từ 4 lên 8 trước vì model nhỏ và GPU
còn VRAM, sau đó đo lại goodput; nếu P95 vẫn vượt SLO, cần admission control thay vì tiếp
tục nhận thêm request.
