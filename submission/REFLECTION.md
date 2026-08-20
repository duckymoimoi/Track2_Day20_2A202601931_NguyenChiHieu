# Báo cáo cá nhân: Bài thực hành ngày 20

Các số liệu trong báo cáo được đo trên máy cá nhân. Tôi chỉ so sánh các cấu hình trong
cùng một môi trường chạy.

Họ tên: Nguyễn Chí Hiếu (2A202601931)

Lớp: K3

Ngày nộp: 2026-08-20

---

## 1. Phần cứng và môi trường chạy *(mục 1, 2; 10 điểm)*

- Hệ điều hành: Windows 11 (AMD64)
- Bộ xử lý: Intel Core i9-11900H thế hệ 11, xung nhịp 2,50 GHz
- Số nhân: 8 nhân vật lý, 16 luồng xử lý
- Tập lệnh CPU: AVX2
- Bộ nhớ: 15,8 GB
- Bộ tăng tốc: NVIDIA GeForce RTX 3060 Laptop, 6144 MiB; CUDA hoạt động
- Gói llama.cpp: `llama-b10488-bin-win-cuda-12.4-x64.zip`
- Mô hình: Qwen3.5 0.8B (`LAB_MODEL=qwen35-0.8b`)
- Mức lượng tử hóa: `Q4_K_M` và `UD-Q2_K_XL`

Nơi chạy: máy tính xách tay cá nhân, không dùng máy chủ đám mây.

### Quá trình thiết lập

Tôi chọn Qwen3.5 0.8B thay cho Gemma để giảm khoảng 4,3 GB dữ liệu tải xuống mà vẫn làm
đủ phần bắt buộc. Giao diện tải của Hugging Face gặp lỗi, vì vậy tôi tải trực tiếp hai
tệp GGUF rồi kiểm tra mã SHA-256. Bản llama.cpp dựng sẵn cho CUDA giúp máy không cần
trình biên dịch hay bộ CUDA Toolkit. PowerShell 5.1 cũng cần thêm dấu BOM và
`PYTHONUTF8=1` để đọc đúng tiếng Việt.

---



## 2. Kết quả đo *(mục 3, 4, 5; 20 điểm)*


| Mức lượng tử hóa | Dung lượng (GB) | Nạp (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | Tổng P50/P95/P99 (ms) | Tốc độ sinh (token/s) |
| ------------ | --------- | --------- | ----------------- | ----------------- | -------------------- | -------------- |
| Q4_K_M       | 0.50      | 12065     | 1171 / 1480       | 13.5 / 17.1       | 2009 / 2318 / 2318   | 74.2           |
| UD-Q2_K_XL   | 0.39      | 9886      | 1123 / 1586       | 15.6 / 21.8       | 2061 / 2712 / 2712   | 64.1           |


### Nhận xét

Trong lần đo cơ sở gồm 10 yêu cầu, bản Q2 chậm hơn 13,6% dù dung lượng nhỏ hơn 22%.
Khi hỏi cùng một câu về thông lượng hữu ích theo SLO, bản Q4 còn nêu được ý về hiệu
năng dưới tải. Bản Q2 trả lời lặp và cho rằng đây là thuật toán của Google. Tôi không
chọn Q2 trên máy này vì chỉ tiết kiệm 0,11 GB nhưng vừa chậm hơn vừa trả lời kém hơn.

---



## 3. Khả năng phục vụ khi có tải *(mục 8, 9, 10; 20 điểm)*


| Số người dùng | Yêu cầu/giây | P50 (ms) | P95 (ms) | P99 (ms) | Số yêu cầu đồng thời hiệu dụng | Tỷ lệ lỗi |
| ----- | ---- | -------- | -------- | -------- | ---------------- | -------- |
| 10    | 1.75 | 4400     | 6800     | 7200     | 8.1              | 0.0%     |
| 50    | 1.85 | 25000    | 27000    | 29000    | 38.6             | 0.0%     |


- Lượng tải đưa vào tăng 5 lần, thông lượng thực chỉ tăng 1,05 lần.
- P95 tăng 3,97 lần.
- Ở mức 50 người dùng, số yêu cầu đồng thời hiệu dụng là 38,6; máy chủ chỉ có 4 khe xử lý.

Giá trị cao nhất của `llamacpp:n_busy_slots_per_decode` khi đo cùng lúc với tải 50 người
dùng là 3,93 trên 4 khe xử lý.

### Nhận xét về điểm bão hòa

Máy chủ bão hòa trước hoặc tại mức 50 người dùng. Số yêu cầu mỗi giây chỉ tăng 1,05 lần,
nhưng P95 tăng 3,97 lần và đạt 27 giây. Kết quả 3,93 trên 4 khe đang bận, kèm 46 yêu cầu
bị hoãn, cho thấy thời gian tăng thêm chủ yếu nằm ở hàng đợi. Với mục tiêu P95 dưới 7
giây, tôi sẽ thử tăng `--parallel` từ 4 lên 8 rồi đo lại. Nếu vẫn không đạt, tôi sẽ giới
hạn lượng yêu cầu nhận vào.

---



## 4. Tích hợp các thành phần *(mục 12, 13; 15 điểm)*


| Ngày | Thành phần | Thật hay mô phỏng? |
| --- | ----------------- | ---------------------------------------------- |
| N16 | Hạ tầng đám mây | Mô phỏng; chạy tại máy cá nhân, không gọi dịch vụ đám mây |
| N17 | Luồng dữ liệu | Mô phỏng; không có công việc nạp và biến đổi dữ liệu |
| N18 | Kho dữ liệu | Mô phỏng; dùng `TOY_DOCS` trong bộ nhớ |
| N19 | Véc-tơ và đặc trưng | Mô phỏng; đối sánh từ khóa, không gọi kho véc-tơ |
| N20 | Phục vụ mô hình | Thật; gọi `llama-server` qua HTTP |


### Thời gian trung bình của ba câu hỏi

- Tạo biểu diễn: 0,0 ms
- Tìm ngữ cảnh: 0,1 ms
- Sinh câu trả lời: 7715,8 ms
- Tổng thời gian: 7716,0 ms

### Nhận xét

Phần sinh câu trả lời chiếm gần như toàn bộ thời gian. Điều này hợp lý vì hai bước tạo
biểu diễn và tìm ngữ cảnh chỉ là mô phỏng trong bộ nhớ. Muốn giảm một nửa thời gian, tôi
sẽ hạ `max_tokens` và thêm điều kiện dừng. Câu hỏi đầu sinh đủ 200 token nên mất 16 giây;
tối ưu bước tìm kiếm chỉ dài 0,1 ms sẽ không tạo khác biệt đáng kể.

---



## 5. Thay đổi có tác động lớn nhất *(mục 11; 10 điểm)*

Tôi tăng số luồng xử lý từ `-t 8` lên `-t 16` khi đo tốc độ sinh `tg128`.

```
trước: 138,7 token/s với -t 8
sau:   162,8 token/s với -t 16
tăng:  1,17 lần
```

### Vì sao tốc độ thay đổi

Kết quả hơi khác dự đoán ban đầu. Tốc độ cao nhất xuất hiện ở 16 luồng xử lý, thay vì
dừng tại 8 nhân vật lý. Với `ngl=99`, toàn bộ các lớp của mô hình đã được chuyển sang RTX
3060. CPU lúc này chủ yếu chuẩn bị lệnh và điều phối CUDA. Cấu hình 16 luồng cấp việc cho
GPU tốt hơn cấu hình 8 luồng trong lần đo này. Ngay cả một luồng cũng đạt 155,5 token/s,
nên GPU mới là phần quyết định tốc độ.

Khi tăng lên 32 luồng, tốc độ giảm còn 159,3 token/s. Số luồng vượt quá khả năng xử lý
logic làm CPU phải chuyển đổi công việc nhiều hơn, trong khi sức tính toán và băng thông
bộ nhớ của GPU không tăng. Mức tăng 1,17 lần là kết quả trên đúng máy này. Tôi vẫn muốn
đo lại vài lần khi máy hoàn toàn rảnh trước khi coi 16 luồng là cấu hình cố định.

---



## 6. Phần mở rộng *(không bắt buộc, tối đa 20 điểm)*

Tôi chọn B2, quét số lớp của mô hình được chuyển sang GPU. Đây là phần mở rộng phù hợp
nhất với máy có RTX 3060 và không cần tải thêm mô hình hay thư viện. Mỗi mức được đo 5
lần với cùng mô hình Q4, 8 luồng CPU và phép đo sinh 128 token.

| `-ngl` | 0 | 8 | 16 | 24 | 32 | 99 |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Tốc độ (token/s) | 27,3 | 51,5 | 82,0 | 163,1 | 211,7 | 209,4 |

```
trước: 27,3 token/s với -ngl 0
sau:   211,7 token/s với -ngl 32
tăng:  7,75 lần
```

Kết quả cho thấy chuyển toàn bộ lớp sang GPU là phù hợp nhất trên máy này. Tốc độ tăng
dần đến `-ngl 32`; `-ngl 99` không tăng thêm vì lúc đó mô hình đã hết lớp để chuyển.
Giá trị 99 là yêu cầu chuyển tối đa, không phải số lớp thực tế của mô hình.

Điểm giới hạn ở đây không phải dung lượng VRAM. Tệp Q4 chỉ khoảng 0,50 GB, trong khi
RTX 3060 có 6 GB VRAM. Với `-ngl 8` hoặc `-ngl 16`, CPU vẫn xử lý một phần mạng và dữ
liệu còn phải qua lại giữa bộ nhớ máy với GPU. Khi gần như toàn bộ trọng số nằm trên
GPU, việc truyền qua ranh giới này giảm và khả năng tính song song của CUDA được tận
dụng tốt hơn. Kết quả chi tiết nằm trong
`benchmarks/bonus-gpu-offload-sweep.md`.

---



## 7. Điều làm tôi ngạc nhiên nhất *(không bắt buộc)*

Mức lượng tử hóa thấp hơn không tự động chạy nhanh hơn. Bản Q2 vừa chậm hơn trong lần đo
cơ sở, vừa trả lời kém hơn. Số luồng tốt nhất cũng không dừng ở số nhân vật lý vì phần
lớn phép tính đã chạy trên GPU.

---



## 8. Tự kiểm tra trước khi đẩy lên GitHub

- [x] Đã đưa `hardware.json` vào Git
- [x] Đã đưa `models/active.json` vào Git
- [x] Đã đưa `benchmarks/01-quickstart-results.md` vào Git (`make bench`)
- [x] Đã đưa `benchmarks/01-tuning-tg128.md` vào Git (`make tune`)
- [x] Đã đưa `benchmarks/02-server-results.md` vào Git (`make load-report`)
- [x] Đã đưa `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` vào Git (`make metrics`)
- [x] Đã đưa `benchmarks/locust-10_stats.csv` và `locust-50_stats.csv` vào Git
- [x] Đã đưa `benchmarks/03-integration-results.md` vào Git (`make pipeline`)
- [x] Mọi phần cần trả lời trong các tệp `benchmarks/*.md` đã có nhận xét
- [x] Có đủ 5 ảnh trong `submission/screenshots/`
- [x] `make verify` trả mã 0
- [x] Kho mã GitHub ở chế độ công khai
- [ ] Đã dán đường dẫn công khai vào VinUni LMS
- [x] Không đưa `models/*.gguf` hay `runtime/` vào Git (đã có trong `.gitignore`)

Kho mã phải để ở chế độ công khai cho đến khi có điểm. Nếu để riêng tư, người chấm sẽ
không xem được bài.
