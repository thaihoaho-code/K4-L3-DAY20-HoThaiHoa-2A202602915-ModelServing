# 02 - Continuous batching under load (u50)

Host `Windows-AMD64` · `--parallel 4` · 7 samples over
60s at 2.0s intervals · raw CSV: `02-server-metrics-u50.csv`

| Gauge | Peak observed |
|:--|--:|
| `n_busy_slots_per_decode` (avg/decode) | 3.99 of 4 slots (100%) |
| `requests_processing` | 4 |
| `requests_deferred` | 46 |
| `kv_cache_usage_ratio` | n/a — not exported by llama.cpp `b10488` |
| `tokens_predicted_total` (final) | 1880 |

Highest sampled value was **3.99 of 4** slots. Note this gauge is llama.cpp's *average* busy slots per decode step, so the number below is the highest average we sampled, not an instantaneous maximum batch width. A peak near 1 means
requests were served one at a time -- either the load was too light to overlap, or
they arrived too far apart. A peak approaching `--parallel` means the scheduler was
genuinely packing concurrent requests into shared decode steps.
`requests_deferred` went above zero: more requests arrived than there were slots, so some waited. That wait is the queue time in your P95.

## Nhận xét của tôi

Batch width trung bình cao nhất được ghi nhận là 3,99/4, gần như toàn bộ decode slot đều hoạt động cùng lúc; cùng thời điểm có 4 request đang xử lý và 46 request bị defer. Con số effective concurrency 47,4 trong báo cáo load test không có nghĩa là 47 request cùng được giải mã: nó tính cả request đang chờ trong hàng đợi theo công thức Little's Law. Vì vậy, metric server đáng tin hơn để xác định batch width đang chạy, còn effective concurrency thể hiện tổng mức chiếm dụng gồm cả hàng đợi.
