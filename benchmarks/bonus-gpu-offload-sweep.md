# Phần mở rộng B2 - Quét số lớp chuyển sang GPU

Máy `Windows-AMD64` · bộ xử lý `nvidia_cuda, vulkan` ·
llama.cpp `b10488` · `threads=8` · phép đo `tg128` · 5 lần lặp

| `-ngl` | `tg128` (token/s) | So với chỉ dùng CPU | So với mức tốt nhất |
|:--|--:|--:|--:|
| 0 | 27.3 | 1.00x | 13% |
| 8 | 51.5 | 1.89x | 24% |
| 16 | 82.0 | 3.00x | 39% |
| 24 | 163.1 | 5.97x | 77% |
| 32 | 211.7 | 7.75x | 100% |
| 99 | 209.4 | 7.67x | 99% |

Mức tốt nhất: `-ngl 32`, đạt 211,7 token/s, nhanh gấp 7,75 lần so với
`-ngl 0` chỉ dùng CPU.

## Kết luận

Chuyển toàn bộ lớp sang GPU là lựa chọn tốt nhất trên máy này. Tốc độ tăng đều từ
27,3 token/s ở `-ngl 0` lên 211,7 token/s ở `-ngl 32`. Mức `-ngl 99` đạt 209,4
token/s, chỉ thấp hơn 1,1%, nên có thể xem là cùng một vùng ổn định trong sai số đo.
Giá trị 99 chỉ yêu cầu llama.cpp chuyển nhiều lớp nhất có thể, không có nghĩa mô hình
thực sự có 99 lớp.

Đường đo phẳng từ 32 đến 99 vì mô hình đã hết lớp để chuyển, không phải vì hết VRAM.
Tệp Q4 chỉ khoảng 0,50 GB, nhỏ hơn nhiều so với 6 GB VRAM của RTX 3060. Ở các mức
`-ngl 8` và `-ngl 16`, CPU vẫn phải tính một phần mạng và dữ liệu phải đi qua ranh giới
giữa bộ nhớ máy và GPU, vì vậy tốc độ mới đạt 51,5 và 82,0 token/s. Khi gần như toàn bộ
trọng số nằm trên GPU, phần truyền qua lại giảm và khả năng tính song song của CUDA được
tận dụng tốt hơn; tốc độ tăng mạnh ở `-ngl 24` rồi đạt đỉnh tại `-ngl 32`.

```
trước: 27,3 token/s với -ngl 0
sau:   211,7 token/s với -ngl 32
tăng:  7,75 lần
```
