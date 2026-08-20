# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.1 | 16012.7 | 16013.0 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.1 | 3502.6 | 3502.7 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.1 | 3632.0 | 3632.2 |

Mean per stage (ms): embed **0.0** · retrieve **0.1** ·
llm **7715.8** · total **7716.0**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Based on the provided context, **Goodput is more useful than raw throughput** because it focuses on **sustaining performance under load** rather than just measuring the maximum instantaneous capacity.

Here is the breakdown based on the context:

1.  **SLO Compliance (Goodput):** The context states that Goodput counts requests per second meeting specific targets (TTFT and TPOT). This means it meas

**What problem does PagedAttention actually solve?**

> Based on the provided context, **PagedAttention** solves the problem of **internal fragmentation in GPU memory**.

The context explicitly states that PagedAttention "stores the KV cache in non-contiguous pages, removing the internal fragmentation that wasted most GPU memory."

**When does splitting prefill and decode help?**

> Based on the provided context, splitting prefill and decode helps when **prefill is compute-bound and decode is memory-bandwidth-bound**.

The context explains that prefilling the model requires processing data that is not available in memory (computational cost), while decoding requires processing data that fits in memory (memory bandwidth cost). By splitting these operations into separate pools,


## Which N16-N19 pieces are real

- **N16 Cloud/IaC:** stubbed; pipeline chạy local và không gọi hạ tầng cloud.
- **N17 Data pipeline:** stubbed; không có ingestion/transform job thực trong lần chạy này.
- **N18 Lakehouse:** stubbed; context đến từ `TOY_DOCS`, không đọc lakehouse.
- **N19 Vector + features:** stubbed; dùng keyword overlap, không gọi vector/feature store.
- **N20 Serving:** real; cả ba câu hỏi gọi `llama-server` qua HTTP.

LLM chiếm 7715.8/7716.0 ms, gần 100% tổng thời gian, đúng với kỳ vọng khi embed/retrieve
chỉ là stub trong bộ nhớ. Muốn giảm latency 2x, tôi sẽ giảm `max_tokens`/thêm stop condition
trước: query đầu chạm đủ 200 output token và mất 16 s, nên cắt phần sinh dài tác động trực
tiếp hơn tối ưu retrieval 0.1 ms.
