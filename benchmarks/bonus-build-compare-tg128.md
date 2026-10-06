# Bonus B1 - So sánh bản prebuilt và bản tự build

Host `Darwin-arm64` · CPU `Apple M1`
Tập lệnh vector phát hiện được: NEON
Hai phía cùng dùng llama.cpp `b10488` · `threads=8` ·
**cùng cố định `ngl=0`** để cô lập ảnh hưởng compiler ·
metric `tg128`, lặp 3 lần

| Binary | Mục tiêu build | tg128 (tok/s) | Tương đối |
|:--|--:|--:|--:|
| bản phát hành prebuilt | CPU dispatch runtime | 26.4 | 1.00x |
| bản tự build | CPU hiện tại (`-DGGML_NATIVE=ON`) | 23.8 | 0.90x |

Trên máy này, binary prebuilt **nhanh hơn 1.11x**.

trước:   26.4 tok/s (bản phát hành prebuilt)
sau:     23.8 tok/s (bản tự build, -DGGML_NATIVE=ON)
tăng tốc: 0.90x

Cùng revision mã nguồn, model, backend và `-ngl`; khác biệt duy nhất là những giả định
compiler được phép dùng về CPU.


### Giá trị riêng của GPU offload trên cùng binary

`tg128` on the source build at `-ngl 99` instead of `-ngl 0`:

| Bản tự build | tg128 (tok/s) | So với lần chạy CPU |
|:--|--:|--:|
| `-ngl 0` (CPU) | 23.8 | 1.00x |
| `-ngl 99` (offload sang MTL0: Apple M1) | 56.9 | 2.39x |

Con số này **không** thuộc phép so sánh B1 phía trên vì đây là knob khác. Tách riêng kết
quả giúp tránh đánh đồng ảnh hưởng của compiler flag với ảnh hưởng của bộ tăng tốc.


## Giải thích

Bản native không thắng mà chậm hơn 10% dù bật NEON, dot-product và FP16 vector arithmetic
riêng cho M1. Bản arm64 prebuilt đã dùng runtime dispatch tối ưu và Accelerate, nên
`-mcpu=native` chỉ thêm ít lợi ích cho workload sinh token nặng về memory; khác biệt
compiler, bố trí code và dao động nhiệt có thể lớn hơn lợi ích tập lệnh nhỏ đó. Kết quả
Metal lớn hơn nhiều (2.39x) vì offload thay đổi tài nguyên thực thi và mức song song bộ
nhớ, không chỉ thay đổi giả định của compiler. Vì vậy không được trình bày số GPU như
speedup compiler của B1.
