# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · host `Windows-AMD64` · llama.cpp `b10488`
Settings: `threads=2` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 17229 | 3339 / 3690 | 84.2 / 96.8 | 8629 / 9138 / 9138 | 11.9 |
| UD-Q2_K_XL | 0.39 | 39800 | 4128 / 5136 | 1232.9 / 1260.8 | 81186 / 84533 / 84533 | 0.8 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **14.88x SLOWER** than `Q4_K_M` here, despite being 0.11 GB smaller. That is a real result, not a mistake: fewer bits only buys speed when decode is limited by memory bandwidth. On a machine that is compute-limited instead — few cores, no GPU offload — the extra dequantization work of a heavily-quantized format can cost more than the bytes it saves. Say which case yours is.

## Nhận xét của tôi

`UD-Q2_K_XL` chỉ nhỏ hơn `Q4_K_M` 0,11 GB (khoảng 22%), nhưng chạy chậm hơn
đáng kể trên máy Windows AMD64 này. Tốc độ giải mã giảm từ 11,9 xuống 0,8 token/giây,
tức chậm hơn khoảng 14,9 lần; thời gian hoàn thành trung vị tăng từ 8,6 giây lên
81,2 giây. Q2 cũng mất nhiều thời gian tải model hơn (39,8 giây so với 17,2 giây).
Vì vậy, mức tiết kiệm dung lượng nhỏ này không đáng đổi lấy tốc độ thấp khi dùng
tương tác trên máy của tôi; Q4 là lựa chọn thực tế hơn. Benchmark này chưa đo chất
lượng câu trả lời, nên cần chạy cả hai model với cùng một câu hỏi trong `serve` để
so sánh thêm.
