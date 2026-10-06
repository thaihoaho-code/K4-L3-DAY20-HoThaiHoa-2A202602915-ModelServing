# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.2 | 17684.8 | 17685.1 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.2 | 10967.5 | 10967.9 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.1 | 0.2 | 24247.6 | 24248.0 |

Mean per stage (ms): embed **0.0** · retrieve **0.2** ·
llm **17633.3** · total **17633.7**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Based on the context provided, **Goodput** is more useful than raw throughput because it is designed specifically to account for **SLOs** (Service Level Objectives).

While raw throughput measures the total requests per second (requests/sec) without considering targets, Goodput only counts requests that met the TTFT (Time-to-Failure) and TPOT (Time-to-Poll/Throughput) targets. This ensures that th

**What problem does PagedAttention actually solve?**

> PagedAttention solves the problem of **internal fragmentation** in GPU memory.

By storing the KV cache in non-contiguous pages, it removes the wasted space that would otherwise be occupied by contiguous blocks of memory.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps when the **model's compute-bound operations (prefill) and memory-bound operations (decode) are separated by a significant latency or bandwidth bottleneck**, such as a large model size, a very long sequence length, or a specific hardware architecture (e.g., GPU vs. CPU).

Specifically, this optimization is most effective when:
1.  **Prefill is the bottleneck:** Th


## N16–N19: thành phần thật và stub

- **N16 Cloud/IaC — stub:** pipeline chạy trên `localhost`; chưa kết nối cluster hay
  stack triển khai cloud.
- **N17 Data pipelines — stub:** dữ liệu được nạp trực tiếp trong bộ nhớ, chưa có DAG
  hoặc batch job.
- **N18 Lakehouse — stub:** tài liệu thử nghiệm nằm trong `TOY_DOCS` dạng dict, chưa
  đọc từ Delta/Iceberg hay lakehouse khác.
- **N19 Vector + features — stub:** truy xuất dùng keyword overlap trên `TOY_DOCS`,
  chưa dùng vector index hay feature store. Kết quả cũng xác nhận backend là
  `keyword overlap`, với embed gần như 0 ms.

LLM là stage áp đảo, chiếm gần như 100% tổng latency trung bình (17.633 ms); điều
này phù hợp với kỳ vọng vì truy xuất trên bộ dữ liệu nhỏ chỉ mất khoảng 0,2 ms.
Muốn giảm latency pipeline khoảng một nửa, tôi sẽ tập trung vào stage LLM: trước
tiên giảm độ dài câu trả lời tối đa để bớt thời gian decode, đồng thời rút gọn prompt
và context để giảm prefill. Đây là nơi có phần lớn thời gian có thể tiết kiệm; tối ưu
embed hoặc retrieval hiện tại gần như không ảnh hưởng tổng thời gian.
