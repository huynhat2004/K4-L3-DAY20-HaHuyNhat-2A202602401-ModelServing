# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Hà Huy Nhật
**MSSV:** 2A202602401
**Cohort:** A20-K4
**Ngày submit:** 2026-10-06

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** macOS 15.5 (Darwin 24.5.0, arm64)
- **CPU:** Apple M1
- **Cores:** 8 physical / 8 logical
- **CPU extensions:** NEON
- **RAM:** 16 GB
- **Accelerator:** Apple Metal
- **llama.cpp asset đã tải:** `llama-b10488-bin-macos-arm64.tar.gz`
- **Model đã dùng:** Qwen3.5 0.8B (`LAB_MODEL=qwen35-0.8b`)
- **Quantization:** Q4_K_M + UD-Q2_K_XL

**Chạy ở đâu:** laptop của tôi
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Probe ban đầu đọc RAM thành 0 GB vì `sysctl` bị sandbox chặn; tôi thêm fallback
`system_profiler`. Python 3.15 cũng cần CA bundle từ `certifi`. Gemma tải quá chậm qua
Xet, nên tôi dùng model Qwen được lab hỗ trợ và tắt Xet để tải HTTP có resume.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 3139 | 108 / 235 | 30.8 / 37.2 | 1375 / 2467 / 2467 | 32.5 |
| UD-Q2_K_XL | 0.39 | 2038 | 79 / 90 | 16.6 / 19.1 | 1131 / 1289 / 1289 | 60.4 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

Q2 nhỏ hơn 22% và decode nhanh hơn 1.86×, nhưng không đáng dùng: với cùng câu hỏi,
Q4 gắn đúng goodput với SLO, còn Q2 mô tả sai thành định dạng output dạng danh sách.
Tôi chọn Q4 vì tiết kiệm 0.11 GB không bù được lỗi ngữ nghĩa quan sát được.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 1.29 | 6300 | 10000 | 13000 | 8.5 | 0.0% |
| 50 | 1.42 | 29000 | 36000 | 40000 | 34.8 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** 1.10×
- **P95 tăng:** 3.60×
- **Effective concurrency ở 50 users:** 34.8 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): 3.93 / 4 slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Server bão hòa rõ ở 50 users: RPS chỉ tăng 1.10× nhưng P95 tăng 3.60×; 3.93/4 slot
bận và 46 request deferred. Effective concurrency 34.8 gồm cả hàng đợi, nên latency
thêm chủ yếu là queue time. Với SLO P95 10 s, tôi tăng `--parallel` trước nếu RAM cho
phép vì nó thêm slot; tăng CPU thread không giúp workload Metal-bound này.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | local process only | stub |
| N17 Data pipeline | in-memory sample documents | stub |
| N18 Lakehouse | no external lakehouse | stub |
| N19 Vector + features | keyword overlap | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.0 ms
- llm: 2129.2 ms
- **stage chiếm nhiều nhất:** llm (100% của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

LLM là bottleneck đúng kỳ vọng vì retrieval stub chạy trong memory. Muốn giảm 2×, tôi
sẽ giới hạn output, giữ prefix ổn định để tận dụng cache và thử quantization sau quality
gate. Tối ưu retrieval gần 0 ms không thể tạo cải thiện đáng kể ở pipeline đã đo.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** hạ `-t` từ 16 xuống 1 cho `tg128` với Metal offload

```
before:  54.2 tok/s (`-t 16`)
after:   58.4 tok/s (`-t 1`)
speedup: 1.08×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

_Giải thích như đang nói với bạn ngồi cạnh. Bám vào **cơ chế**, không phải "vibes":
memory bandwidth? vector width? cache residency? scheduling? queueing? Nếu kết quả
**khác** với kỳ vọng từ deck — nói rõ, và giải thích vì sao. Grader thưởng điểm cho
lập luận đúng về một kết quả bất ngờ, hơn là một con số đẹp không được giải thích._

Đỉnh nằm ở một thread thay vì tám physical core. Với `ngl=99`, gần như toàn bộ layer
đã chạy trên Metal, nên tăng CPU thread không mở rộng phần decode chi phối. Các thread
host bổ sung chỉ tăng scheduling/synchronization và cùng tranh unified-memory bandwidth.

Spread chỉ 1.08× cũng quan trọng: workload bị GPU và memory bandwidth chi phối, không
phải CPU parallelism. Vì vậy 16 thread chậm hơn dù trực giác CPU-only thường kỳ vọng
throughput tăng tới gần số physical core. Kết quả này cũng giải thích vì sao tăng thread
không phải knob đầu tiên để sửa saturation trong load test.

---

## 6. Bonus  *(optional — tối đa 10 điểm)*

> Bỏ trống nếu không làm. Xem `docs/bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** B1 build-compare; B2 context sweep; B3 before/after; B4 C5 smallest useful
quantization; B5 C8 semantic cache offline

**Numbers:**

```
before:  8424 ms TTFT contribution (8192 prompt tokens)
after:   5049 ms TTFT contribution (4096 prompt tokens)
speedup: 1.67× lower prefill latency
```

**Điều này nói lên gì mà deck chưa nói:**

Native CPU build đạt 23.8 tok/s, chậm hơn prebuilt 26.4 tok/s (0.90×), cho thấy prebuilt
arm64 đã tối ưu tốt và workload memory-heavy không tự động hưởng lợi từ `-mcpu=native`.
Ngược lại, Metal offload trên cùng native binary đạt 56.9 tok/s, nhanh 2.39× CPU run.
Context sweep bẻ cong mạnh tại 4096 token: 5049 ms, 1.35× dự đoán tuyến tính. Vì thế
giảm context có giá trị trực tiếp hơn compile flag trên máy này.

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

Điều bất ngờ nhất là một thread nhanh hơn 16 thread khi model offload sang Metal, và
native build lại thua prebuilt. Cả hai nhắc tôi xác định bottleneck bằng số đo trước khi
áp dụng quy tắc tối ưu từ workload CPU-only.

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

Sử dụng OpenAI Codex để đọc yêu cầu, chạy và chẩn đoán script, sửa lỗi probe/cổng,
tổ chức benchmark, đối chiếu số liệu và hỗ trợ soạn báo cáo. Mọi con số trong báo cáo
đều lấy từ các lần chạy thật trên máy này; nhận xét chất lượng được kiểm tra bằng cùng
prompt trên hai quantization.
