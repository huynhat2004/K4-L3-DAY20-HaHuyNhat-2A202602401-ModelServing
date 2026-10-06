# 02 - Serving: load test và phân tích bão hòa

Host `Darwin-arm64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=8` ·
`ngl=99`

| Người dùng | Request | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Concurrency hiệu dụng | Tỷ lệ lỗi |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 71 | 1.29 | 6300 | 10000 | 13000 | 8.5 | 0.0% |
| 50 | 83 | 1.42 | 29000 | 36000 | 40000 | 34.8 | 0.0% |

*Concurrency hiệu dụng = RPS x latency trung bình (Little's Law), tức số request thực sự
in-flight bất kể Locust mô phỏng bao nhiêu người dùng. Chỉ số này gồm cả request trong hàng
đợi nên tỷ lệ occupancy/slot có thể lớn hơn 1.0. Để đo mức sử dụng slot thật, dùng gauge
của server (`make metrics`).*

## Hai lần chạy cho thấy điều gì

| Khi tăng từ 10 lên 50 người dùng | |
|:--|--:|
| Tải đưa vào | 5x |
| Throughput thực nhận | **1.10x** (22% mức tuyến tính) |
| Latency P95 | **3.60x** |
| Concurrency hiệu dụng ở 50 người dùng | 34.8 so với `--parallel 4` slot (tỷ lệ occupancy/slot 8.70) |

**Đã bão hòa.** Throughput chỉ tăng 1.10x khi tải đưa vào tăng 5x, còn concurrency hiệu
dụng 34.8 vượt xa 4 decode slot. Hệ thống bão hòa ở đâu đó không quá 50 người dùng; tải
thêm sau điểm đó chủ yếu trở thành thời gian chờ thay vì throughput.

Throughput tăng 1.10x trong khi P95 tăng 3.60x. Khoảng cách này là lập luận về goodput:
sau bão hòa, throughput tăng rất ít nhưng phải đánh đổi nhiều latency; nếu SLO đặt theo
P95 thì các request thêm vào không còn được phục vụ trong giới hạn đó.

## Phân tích

Server đã gần bão hòa ở 10 người dùng và bão hòa rõ tại 50: tải đưa vào tăng 5x nhưng
RPS chỉ tăng 1.10x, còn P95 tăng 3.60x lên 36 giây. Tại 50 người dùng, 34.8 request
in-flight hiệu dụng tranh bốn decode slot; metrics cho thấy 3.93/4 slot bận và 46 request
bị hoãn, nên phần lớn latency tăng thêm là queueing. Với SLO P95 10 giây, tôi sẽ tăng
`--parallel` trước nếu bộ nhớ cho phép rồi đo lại; thêm CPU thread không giảm decode bị
giới hạn bởi Metal và còn chậm hơn trong thread sweep.
