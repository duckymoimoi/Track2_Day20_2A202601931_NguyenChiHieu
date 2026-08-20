# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · host `Windows-AMD64` · llama.cpp `b10488`
Settings: `threads=8` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 12065 | 1171 / 1480 | 13.5 / 17.1 | 2009 / 2318 / 2318 | 74.2 |
| UD-Q2_K_XL | 0.39 | 9886 | 1123 / 1586 | 15.6 / 21.8 | 2061 / 2712 / 2712 | 64.1 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.16x SLOWER** than `Q4_K_M` here, despite being 0.11 GB smaller. That is a real result, not a mistake: fewer bits only buys speed when decode is limited by memory bandwidth. On a machine that is compute-limited instead — few cores, no GPU offload — the extra dequantization work of a heavily-quantized format can cost more than the bytes it saves. Say which case yours is.

## My observation

`UD-Q2_K_XL` nhỏ hơn 0.11 GB nhưng decode chậm hơn `Q4_K_M` khoảng 13.6% (64.1 so với
74.2 tok/s), đồng thời P95 E2E cao hơn. Với cùng câu hỏi về goodput@SLO, Q4 còn nêu
được ý về hiệu năng dưới tải, trong khi Q2 lặp lại và bịa rằng đây là thuật toán của
Google. Vì vậy phần tiết kiệm 22% dung lượng không đáng đổi lấy tốc độ và chất lượng.
