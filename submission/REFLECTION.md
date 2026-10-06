# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Hồ Thái Hòa
**MSSV:** 2A202602915
**Cohort:** _<A20-K4>_
**Ngày submit:** 2026-10-06

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** Windows 11 (AMD64)
- **CPU:** Intel Core i5-7200U @ 2.50 GHz
- **Cores:** 2 physical / 4 logical
- **CPU extensions:** chưa được `hardware.json` ghi nhận trên Windows
- **RAM:** 15.9 GB
- **Accelerator:** Vulkan (phát hiện thiết bị Vulkan)
- **llama.cpp asset đã tải:** `llama-b10488-bin-win-vulkan-x64.zip`
- **Model đã dùng:** Qwen3.5 0.8B (`LAB_MODEL=qwen35-0.8b`)
- **Quantization:** `Q4_K_M` + `UD-Q2_K_XL` (từ `models/active.json`)

**Chạy ở đâu:** máy Windows local
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Tôi chạy `bootstrap.ps1` trên Windows; lab dùng Python 3.13 và llama.cpp Vulkan b10488. Máy có 15.9 GB RAM nên dùng model Qwen3.5 0.8B. Các artifact hiện có cho thấy setup và benchmark đã chạy được; không có workaround riêng nào được ghi lại.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 17229 | 3339 / 3690 | 84.2 / 96.8 | 8629 / 9138 / 9138 | 11.9 |
| UD-Q2_K_XL | 0.39 | 39800 | 4128 / 5136 | 1232.9 / 1260.8 | 81186 / 84533 / 84533 | 0.8 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

Q2 nhỏ hơn khoảng 22% nhưng giải mã chậm hơn khoảng 14.9 lần (0.8 so với 11.9 token/giây), nên không đáng đổi lấy dung lượng tiết kiệm trên máy này. Tôi chưa có kết quả so sánh chất lượng bằng cùng một câu hỏi; benchmark này chỉ đo hiệu năng.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 0.07 | 58000 | 58000 | 58000 | 4.0 | 0.0% |
| 50 | 0.41 | 122000 | 122000 | 122000 | 47.4 | 88.0% |

- **Offered load tăng 5×, throughput thực tăng:** 5.84×
- **P95 tăng:** 2.10×
- **Effective concurrency ở 50 users:** 47.4 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): 3.99 / 4 slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Ở 50 users, server đã chạm giới hạn phục vụ: 3.99/4 slot bận, 46 request bị defer và 44/50 request timeout sau 120 giây. P95 đạt 122 giây. Metric `requests_deferred` cho thấy có hàng đợi, nhưng chưa tách được chính xác bao nhiêu latency là chờ và bao nhiêu là xử lý. Tôi sẽ giảm token đầu ra cho prompt dài trước để giảm thời gian chiếm slot, rồi đo lại goodput và P95.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | localhost, không có cluster/stack cloud trong pipeline | stub |
| N17 Data pipeline | danh sách dữ liệu trong bộ nhớ, không có DAG/batch job | stub |
| N18 Lakehouse | `TOY_DOCS` dạng dict, không có Delta/Iceberg | stub |
| N19 Vector + features | keyword overlap, không có vector index/feature store | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.2 ms
- llm: 17633.3 ms
- **stage chiếm nhiều nhất:** LLM (xấp xỉ 100% của total 17633.7 ms)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

LLM chiếm gần như toàn bộ latency; đây là điều tôi dự đoán vì keyword retrieval trên bộ dữ liệu nhỏ chỉ mất 0.2 ms. Để giảm latency khoảng 2×, tôi sẽ giảm output token tối đa và rút gọn prompt/context nhằm giảm cả decode lẫn prefill. Tối ưu embed/retrieval hiện tại hầu như không làm thay đổi tổng thời gian.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** giảm số thread từ `-t 2` (mặc định theo số lõi vật lý) xuống `-t 1`

```
before:  13.84 tok/s (`-t 2`)
after:   13.93 tok/s (`-t 1`)
speedup: 1.01× (tăng khoảng 0.65%)
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

_Giải thích như đang nói với bạn ngồi cạnh. Bám vào **cơ chế**, không phải "vibes":
memory bandwidth? vector width? cache residency? scheduling? queueing? Nếu kết quả
**khác** với kỳ vọng từ deck — nói rõ, và giải thích vì sao. Grader thưởng điểm cho
lập luận đúng về một kết quả bất ngờ, hơn là một con số đẹp không được giải thích._

Với `ngl=99`, phần lớn model được offload qua backend Vulkan, nên tăng thread CPU từ 1 lên 2 hoặc 4 gần như không cải thiện decode: `-t 2` và `-t 4` đều đạt 13.84 tok/s, còn `-t 1` đạt 13.93 tok/s. Mức tăng chỉ khoảng 0.65%, nên có thể nằm trong sai số giữa các lần đo chứ chưa phải cải thiện đáng kể.

Kết quả cho thấy trên cấu hình này thêm thread không làm GPU giải mã nhanh hơn; các thread CPU có thể tranh thời gian chạy trên số lõi vật lý giới hạn và tạo thêm chi phí điều phối. Đây là phép đo `tg128` của llama-bench, nên kết luận chỉ áp dụng cho phép thử decode đó; cần đo lại toàn pipeline nếu muốn khẳng định latency thực tế thay đổi.

---

## 6. Bonus  *(optional — tối đa 10 điểm)*

> Bỏ trống nếu không làm. Xem `docs/bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _<B1 build-compare / B2 sweep nào / B4 challenge nào / B5 lựa chọn nào>_

**Numbers:**

```
before:  <số>
after:   <số>
speedup: <X.Y>×
```

**Điều này nói lên gì mà deck chưa nói:**

_(để trống nếu bạn không làm phần này)_

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

---

## 8. Self-check trước khi push

- [ ] `hardware.json` committed
- [ ] `models/active.json` committed
- [ ] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [ ] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [ ] `benchmarks/02-server-results.md` committed (`make load-report`)
- [ ] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [ ] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [ ] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [ ] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [ ] 5 screenshots trong `submission/screenshots/`
- [ ] `make verify` → **exit 0**
- [ ] Repo tên đúng mẫu `K4-L3-DAY20-HoVaTen-MSSV-ModelServing` (xem `docs/SUBMISSION.md`)
- [ ] Repo GitHub ở chế độ **public**
- [ ] Đã push và paste public URL vào VinUni LMS **trước 23:59 (UTC+7) ngày làm lab**
- [ ] **Không** commit `models/*.gguf`, `runtime/` hay `.env` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.

---

## 9. Khai báo sử dụng AI  *(xem `docs/RULES.md` §3)*

_(Công cụ nào, dùng vào việc gì. Ghi "Không dùng" nếu không dùng.)_
