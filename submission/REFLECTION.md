# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** _Trần Quang Trọng - 2A202601461_
**Cohort:** _A20-K3_
**Ngày submit:** _2026-08-20_

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** _Darwin 25.5.0 (macOS, arm64)_
- **CPU:** _Apple M3_
- **Cores:** _8 physical / 8 logical_
- **CPU extensions:** _NEON_
- **RAM:** _16.0 GB_
- **Accelerator:** _Apple Metal_
- **llama.cpp asset đã tải:** _llama-b10488-bin-macos-arm64.tar.gz_
- **Model đã dùng:** _Gemma 4 E2B_ (`LAB_MODEL=`_gemma4-e2b_)
- **Quantization:** _UD-Q4_K_XL_ + _UD-Q2_K_XL_ (từ `models/active.json`)

**Chạy ở đâu:** _laptop của tôi_
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Setup chạy local trên MacBook Apple M3 với 16 GB RAM, dùng llama.cpp prebuilt Metal và
Gemma 4 E2B mặc định. Hai quantization Q4/Q2 đã tải thành công; quá trình tải model
chậm nhưng không cần workaround.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 4157 | 144 / 295 | 23.3 / 25.0 | 1604 / 1855 / 1855 | 43.0 |
| UD-Q2_K_XL | 2.24 | 3085 | 146 / 366 | 19.7 / 19.9 | 1384 / 1620 / 1620 | 50.7 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

UD-Q2_K_XL nhỏ hơn 0,73 GB và decode nhanh hơn 1,18x; load model cũng nhanh hơn.
TTFT P50 gần như tương đương, TPOT thấp hơn rõ rệt. Vì vậy Q2 có lợi cho tốc độ và
dung lượng; chất lượng trả lời cần được kiểm tra bằng cùng một câu hỏi trên cả hai bản.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 1.00 | 8900 | 12000 | 13000 | 8.7 | 0.0% |
| 50 | 1.04 | 29000 | 48000 | 50000 | 29.3 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** _1.05×_
- **P95 tăng:** _4.00×_
- **Effective concurrency ở 50 users:** _29.3_ so với `--parallel` = _4_ slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): _3.99_ / _4_ slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Server bão hòa ở mức 50 users: tải tăng 5× nhưng RPS chỉ tăng 1,05×, P95 tăng 4×,
effective concurrency lên 29,3 so với 4 slots, và peak busy slots là 3,99/4 với 46
request deferred. Phần latency tăng chủ yếu là queue time. Tôi sẽ thử tăng
`--parallel` trước, vì queue do slots bận; sau đó kiểm tra lại RAM và P95.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | localhost-only | stub |
| N17 Data pipeline | in-memory `TOY_DOCS` | stub |
| N18 Lakehouse | toy dict | stub |
| N19 Vector + features | keyword overlap trên `TOY_DOCS` | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: _0.0 ms_
- retrieve: _0.0 ms_
- llm: _1207.3 ms_
- **stage chiếm nhiều nhất:** _llm_ (_100%_ của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

Trong lần chạy này, bottleneck hoàn toàn là `llm` (1207,3 ms trung bình); `embed` và
`retrieve` đều 0,0 ms vì đang dùng keyword-overlap fallback trên dữ liệu toy. Điều này
đúng với kỳ vọng. Nếu cần giảm latency 2×, tôi sẽ tối ưu LLM trước bằng model nhỏ hơn,
quantization phù hợp hoặc giảm output/context tokens.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** _Giảm số thread từ `-t 8` xuống `-t 4`_

```
before:  42.8 tok/s (`-t 8`)
after:   45.3 tok/s (`-t 4`)
speedup: 1.06×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

Kết quả tốt nhất nằm ở `-t 4`, không phải 8 physical cores như dự đoán đơn giản. Từ
4 lên 8, tốc độ giảm từ 45,3 xuống 42,8 tok/s; lên 16 chỉ còn 37,8 tok/s. Điều này
cho thấy decode bị giới hạn bởi memory bandwidth và cache/scheduling hơn là thiếu
compute threads. Thread bổ sung cạnh tranh cùng băng thông bộ nhớ và tạo overhead
scheduling, nên làm throughput giảm. Trên máy này, giảm thread từ 8 xuống 4 cho
speedup thực tế 1,06x.

---

## 6. Bonus  *(optional — tối đa 20 điểm)*

> Bỏ trống nếu không làm. Xem `bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _<B1 build-compare / B2 sweep nào / B4 challenge nào / B5 lựa chọn nào>_

**Numbers:**

```
before:  <số>
after:   <số>
speedup: <X.Y>×
```

**Điều này nói lên gì mà deck chưa nói:**

_(để trống nếu bạn không làm phần này)_

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

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
- [ ] Repo GitHub ở chế độ **public**
- [ ] Đã paste public URL vào VinUni LMS
- [ ] **Không** commit `models/*.gguf` hay `runtime/` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.
