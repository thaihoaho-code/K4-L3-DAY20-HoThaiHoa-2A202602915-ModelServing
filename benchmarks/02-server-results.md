# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=2` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 4 | 0.07 | 58000 | 58000 | 58000 | 4.0 | 0.0% |
| 50 | 50 | 0.41 | 122000 | 122000 | 122000 | 47.4 | 88.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **5.84x** (117% of linear) |
| P95 latency | **2.10x** |
| Effective concurrency at 50 users | 47.4 vs `--parallel 4` slots (occupancy/slot ratio 11.86) |

**At capacity, still scaling.** All 4 decode slots are busy (effective concurrency 47.4) but throughput still rose 5.84x. You are at the knee -- the next increment of load is where P95 starts to run away.

P95 grew no faster than throughput (2.10x vs 5.84x), so this server still has headroom at 50 users.

> **Small sample.** Only 4 requests completed in the
> shorter run, so these percentiles are indicative rather than solid. Note also that
> locust averages only *completed* requests: when the run ends with requests still
> queued, effective concurrency is an **under**-estimate. Trust the throughput-scaling
> row over the concurrency row here, and run longer (`-t 3m`) if you want firmer numbers.

## Nhận xét của tôi

Ở mức 50 người dùng, server đã chạm giới hạn phục vụ thực tế: cả 4 decode slot đều bận, có 46 request bị defer, và 44/50 request (88%) timeout sau 120 giây. P95 cũng đạt 122 giây, tức các request không đáp ứng được một SLO tương tác thông thường. RPS tăng khi tải tăng từ 10 lên 50 người, nhưng đổi lại hàng đợi dài và phần lớn request thất bại; vì vậy đây không phải mức tải vận hành tốt. Tôi sẽ giảm số token đầu ra cho prompt dài trước để mỗi request chiếm slot ít thời gian hơn. Dữ liệu này cho thấy tắc nghẽn ở năng lực xử lý khi slot đầy và request xếp hàng; chưa đủ để kết luận giới hạn cụ thể là băng thông bộ nhớ hay dung lượng KV cache.
