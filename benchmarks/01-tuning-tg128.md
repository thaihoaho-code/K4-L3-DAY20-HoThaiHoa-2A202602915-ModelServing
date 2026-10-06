# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` · host `Windows-AMD64` · llama.cpp `b10488`
CPU: **2 physical · 4 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 13.9 | 100% |
| 2 | 13.8 | 99% |
| 4 | 13.8 | 99% |

**Best**: `-t 1` at 13.9 tok/s
**Slowest tested**: `-t 2` at 13.8 tok/s (1.01x spread)
**Against the physical-core default** (`-t 2`, 13.8 tok/s): 1.01x

Use this in your run:

```bash
LAB_N_THREADS=1 make bench
```

## Giải thích của tôi

Kết quả gần như đi ngang: `-t 1` đạt cao nhất với 13,93 token/giây, còn `-t 2` và
`-t 4` đều đạt 13,84 token/giây. Chênh lệch giữa mức tốt nhất và thấp nhất chỉ
khoảng 0,7%, nên chưa thấy một điểm gãy (knee) rõ rệt hay lợi ích đáng kể khi tăng
số luồng. Với `ngl=99`, phần lớn lớp model được offload lên GPU; vì vậy số luồng CPU
có thể ít ảnh hưởng đến tốc độ giải mã. Thêm luồng cũng có thể tạo thêm chi phí điều
phối mà không giúp tăng thông lượng. Trên máy này, `-t 1` là lựa chọn tốt nhất theo
số đo, nhưng mức cải thiện so với mặc định `-t 2` rất nhỏ và có thể nằm trong sai
số giữa các lần chạy.

Các thread thừa có thể tranh thời gian chạy trên số lõi vật lý giới hạn và cùng sử
dụng băng thông bộ nhớ để chuẩn bị dữ liệu cho GPU. Vì phần lớn model đã chạy trên
GPU, thêm thread CPU không làm GPU giải mã nhanh hơn mà còn phát sinh chi phí điều phối.
