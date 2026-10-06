# 02 - Continuous batching dưới tải (u50)

Host `Darwin-arm64` · `--parallel 4` · 28 mẫu trong
60 giây, khoảng cách 2.0 giây · CSV gốc: `02-server-metrics-u50.csv`

| Gauge | Đỉnh quan sát được |
|:--|--:|
| `n_busy_slots_per_decode` (trung bình/decode) | 3.93/4 slot (98%) |
| `requests_processing` | 4 |
| `requests_deferred` | 46 |
| `kv_cache_usage_ratio` | không có — llama.cpp `b10488` không export |
| `tokens_predicted_total` (cuối) | 9759 |

Giá trị mẫu cao nhất là **3.93/4** slot. Gauge này là số slot bận *trung bình* trên mỗi
decode step, không phải batch width cực đại tức thời. Đỉnh gần 1 nghĩa là request được
xử lý lần lượt; đỉnh tiến gần `--parallel` nghĩa là scheduler thực sự gộp request đồng
thời vào các decode step dùng chung. `requests_deferred` lớn hơn 0 cho thấy request đến
nhiều hơn số slot, nên một phần phải chờ; thời gian đó xuất hiện trong P95.

## Nhận xét

Độ rộng decode trung bình đạt đỉnh 3.93/4 slot, là bằng chứng trực tiếp continuous batching
đã giữ gần như mọi slot hoạt động. Little's Law cho 34.8 request đồng thời hiệu dụng, lớn
hơn bốn vì gồm cả công việc trong hàng đợi; đỉnh 46 request bị hoãn xác nhận cách hiểu này.
Tôi dùng gauge server cho mức sử dụng slot và Little's Law cho tổng request in-flight vì
chúng đo hai phần khác nhau của cùng một hệ thống bão hòa.
