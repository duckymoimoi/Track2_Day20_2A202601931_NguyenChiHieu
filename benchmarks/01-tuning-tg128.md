# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` · host `Windows-AMD64` · llama.cpp `b10488`
CPU: **8 physical · 16 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 155.5 | 96% |
| 4 | 144.7 | 89% |
| 8 | 138.7 | 85% |
| 16 | 162.8 | 100% |
| 32 | 159.3 | 98% |

**Best**: `-t 16` at 162.8 tok/s
**Slowest tested**: `-t 8` at 138.7 tok/s (1.17x spread)
**Against the physical-core default** (`-t 8`, 138.7 tok/s): 1.17x

Use this in your run:

```bash
LAB_N_THREADS=16 make bench
```

## My explanation

Điểm tốt nhất nằm ở 16 thread (162.8 tok/s), không phải 8 physical core. Đây không phải
đường cong CPU decode điển hình vì `ngl=99` đã đưa toàn bộ layer lên RTX 3060; CPU chủ yếu
chuẩn bị công việc và điều phối CUDA. 16 logical thread cấp việc cho GPU tốt hơn cấu hình
8 thread trong lần đo này, nhưng lợi ích không tuyến tính: ngay cả 1 thread đã đạt 155.5
tok/s, cho thấy GPU mới là phần chi phối.

Tăng tiếp lên 32 thread làm tốc độ giảm nhẹ còn 159.3 tok/s. Khi số worker vượt logical
core, context switching và contention ở phần host/driver tăng nhưng không bổ sung năng lực
tính toán cho GPU. Vì thế tôi chọn `-t 16`; speedup 1.17x so với mặc định `-t 8` là kết quả
đo thực tế, còn hình dạng không đơn điệu cũng cho thấy cần benchmark thay vì suy luận chỉ
từ số physical core.
