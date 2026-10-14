MSSV: 2A202602630

> Seed và thị trấn tính từ 5 số cuối `02630`, truyền qua `LAB11_MSSV=02630` (`meta.json: _run.mssv = "02630"`). Mọi số dưới đây lấy từ `evidence/` của lần chạy `LAB11_MSSV=02630 python run_all.py all` (không `--quick`, `meta.json: _run.quick = false`), cộng thêm `run_all.py mine hazard.json` và `run_all.py export` cho bước G. Số nào lấy từ lần chạy thêm ngoài `evidence/` đều ghi rõ *(chạy thêm)*.

# Lab 11 · Scenario Forge — báo cáo

Thị trấn của bạn (`meta.json: _run.town`): **B** (thị trấn "mưa nhiều": `p_bin` = (đường ướt 0,30; ngó qua xe buýt 0,85; phanh sau khi vào làn 0,15), `cut_gap_median` = 12 m).

## 1. Phát lại log có trung thực không? (bước A)

| `replay.json` | giá trị |
|---|---|
| `n` | 400 |
| `dropped_human_crash_logs` (người lái tự va chạm, bỏ) | 1 |
| `n_used` | 399 |
| `replay.crashes` (planner trên log phát lại) | **104**: `rear` 90, `side` 14 |
| `reactive.crashes` (planner trong thế giới phản ứng) | **0** |
| `crash_in_both` | 0 |

Phần lớn va chạm của "phát lại" là `rear` (90/104) vì planner **phanh sớm và đi chậm hơn** người lái: nó dùng `T` = 1,6 s (người 1,1 s) và giảm về `cautionV` = 7,5 m/s khi tới gần vạch qua đường bị xe buýt che. Xe bám đuôi TG trong log chỉ lặp lại đúng quỹ đạo đã ghi khi đi sau **người lái**. Nó không biết xe phía trước giờ đi chậm hơn, nên không phanh và đâm vào đuôi. Ngoài đời, TG sẽ thấy xe ta chậm lại và phanh theo. Vì vậy đây là va chạm **giả**: trong thế giới phản ứng, đúng 399 kịch bản đó cho 0 va chạm, và `crash_in_both` = 0. 14 vụ `side` cùng loại: xe C2 chuyển làn theo đúng thời điểm đã ghi, tức là theo khe trống bên cạnh người lái, nhưng xe ta lúc đó ở vị trí khác. Trình mô phỏng cũng đánh dấu các va chạm này là `fake = true`.

**Ví dụ `replay_only[0]`** (kịch bản 0, `contact = rear`, `min_ttc_reactive` = 2,44 s; trình xem chương 9 chiếu nó là R-01). Đường khô (μ = 0,8), người đi bộ không ngó (`pedLook` = 0). *(chạy thêm, cùng tham số)*: lúc t = 10 s người lái còn chạy 10,2 m/s, còn planner đã giảm xuống 8,3 m/s vì vạch qua đường bị che. Khoảng cách xe ta–TG trên log phát lại là 14,1 → 7,8 → 5,7 m (t = 10, 12, 14 s), còn trong thế giới phản ứng là 16,5 → 12,6 → 11,3 m. TG phát lại đâm đuôi xe ta lúc t = 19,3 s. TG phản ứng thì giữ khoảng cách và không va chạm. Ở thế giới phản ứng, TTC nhỏ nhất của xe ta là 2,44 s, không có va chạm.

## 2. Kịch bản bất khả thi (bước B)

Các luật trong `is_feasible`, mỗi luật dựa trên một giới hạn vật lý:

| Luật | Căn cứ vật lý |
|---|---|
| Gia tốc ngang khi chuyển làn a_lat = (π/cutDur)²·3,5/2 ≤ μg | Lốp chỉ cho lực ngang tối đa μ·m·g (vòng ma sát). Đổi làn 3,5 m trong 1,2 s trên đường khô cần khoảng 12 m/s², vượt 0,8·9,81 = 7,8 m/s², nên xe sẽ trượt chứ không đổi làn được. |
| Phanh sau khi vào làn `cutBrake` ≤ μg | Gia tốc phanh dọc cũng bị ma sát giới hạn: đường ướt μ = 0,5 cho tối đa 4,9 m/s². |
| `pedV` ≤ 2,5 m/s | Người đi bộ thường đi 1,2–1,5 m/s. Trên 2,5 m/s là chạy, không còn là người đi bộ theo mô hình "đi qua vạch". |
| 0 < `cutV` ≤ 22 m/s | Xe phải đang đi tới, và không vượt khoảng 80 km/h trên phố đô thị. |

Số kịch bản bất khả thi bị bỏ (`hunt.json: <kiểu>.infeasible`): `bus_bypass` 4, `cut_in` 2, `crosswalk_block` 0, **`rear_ended` 49**. Natural MC có `mc.json: infeasible` = 17/3000. *(chạy thêm, đọc `cloud.json`)*: **cả 55** kịch bản bất khả thi trong các cuộc săn đều vi phạm luật gia tốc ngang (cutDur quá ngắn so với μ). Không kịch bản nào vi phạm luật phanh, `pedV` hay `cutV`.

**Vì sao CEM bị kéo về đó:** điểm nguy hiểm của `rear_ended` là TTC của TG tới xe ta. Cách dễ nhất để TG suýt đâm là bắt xe ta phanh gấp, và cách dễ nhất để xe ta phanh gấp là cho C2 chuyển làn **thật nhanh** (cutDur rất ngắn) ngay trước mặt, nhất là trên đường ướt (TG cũng không phanh kịp). Những kịch bản này là "nguy hiểm nhất" theo điểm số, nên elite chứa đầy chúng, và trong `hunt.json: rear_ended.trace` số bất khả thi tăng lên 13 ở vòng 4. Nhưng xe thật không thể đổi làn như vậy.

**Nếu đếm chúng là va chạm:** ước lượng IS sẽ cộng xác suất của những sự kiện **không thể xảy ra**, nên bị chệch lên. Đồng thời CEM sẽ "thưởng" vùng bất khả thi và khớp q vào đó, nên cả cuộc săn hội tụ về một kiểu thất bại không tồn tại. Ta sẽ báo một rủi ro `rear_ended` mà xe thật không bao giờ gặp, rồi "sửa" planner cho một tình huống ảo.

## 3. Monte Carlo cho bạn biết gì (bước C1)

`mc.json`: **k = 1**, **n = 3000**, **p̂ = 3,33·10⁻⁴**, Wilson 95 % = **[5,88·10⁻⁵ ; 1,89·10⁻³]**. Vụ duy nhất là `bus_bypass` (`mc.json: modes = {"bus_bypass": 1}`). Khoảng này rộng hơn 30 lần từ đầu này sang đầu kia: MC nói được "khoảng 10⁻⁴ đến 10⁻³", không hơn.

**Cần bao nhiêu lần chạy để còn ±20 % của p:** 1,96·√(p(1−p)/n) ≤ 0,2·p ⟹ n ≈ 1,96²/(0,2²·p) ≈ 96/p = 96/3,33·10⁻⁴ ≈ **288 000 lần chạy** (gần 100 lần ngân sách 3 000). Với 4 ms/kịch bản, con số đó tương đương khoảng 20 phút tính toán thuần, nhưng chỉ cho **một** con số tổng.

**Kiểu chưa hề xuất hiện:** `cut_in`, `crosswalk_block`, `rear_ended` và `other` đều có k = 0 trong 3 000 lần (`mc.json: per_mode`). Con số đúng cần báo **không phải 0** mà là cận trên Wilson: **P < 1,28·10⁻³** (95 %) cho mỗi kiểu (`mc.json: per_mode.<kiểu>.hi = 0.00128`). Khoảng chuẩn ở k = 0 cho [0, 0], tức "không bao giờ", và điều đó sai.

## 4. Importance sampling, ESS và độ suy biến (bước C3)

`estimate.json` (mỗi seed n = 600 mẫu/kiểu, q = 0,8·CEM + 0,2·tự nhiên):

| seed | kiểu | `p` | `hits` | `ess` | `top_share` |
|---|---|---|---|---|---|
| 0 | bus_bypass | 2,18·10⁻³ | 82 | 112 | 0,34 |
| 0 | cut_in | 0 → cận `upper_zero_hits` = 2,5·10⁻² | 0 | 134 | 0 |
| 0 | crosswalk_block | 3,14·10⁻⁷ | 73 | 119 | 0,25 |
| 0 | rear_ended | 0 → cận 2,5·10⁻² | 0 | 123 | 0 |
| 1 | bus_bypass | 3,44·10⁻³ | 89 | 128 | **0,75** |
| 1 | cut_in | 0 → cận 2,5·10⁻² | 0 | 127 | 0 |
| 1 | crosswalk_block | 6,38·10⁻⁷ | 69 | 129 | 0,50 |
| 1 | rear_ended | 0 → cận 2,5·10⁻² | 0 | 129 | 0 |
| 2 | bus_bypass | 5,28·10⁻⁴ | 86 | 131 | 0,18 |
| 2 | cut_in | 0 → cận 2,5·10⁻² | 0 | 137 | 0 |
| 2 | crosswalk_block | 4,75·10⁻⁷ | 71 | 148 | 0,21 |
| 2 | rear_ended | 0 → cận 2,5·10⁻² | 0 | 116 | 0 |
| | **tổng** (`seeds[i].total`) | 2,18·10⁻³ / 3,44·10⁻³ / 5,28·10⁻⁴ | | | |

**Ba seed có đồng ý không?** Chỉ trong cùng bậc độ lớn: tổng chênh **6,5 lần** (5,28·10⁻⁴ đến 3,44·10⁻³), trung vị 2,18·10⁻³. `crosswalk_block` ổn định hơn (3,1–6,4·10⁻⁷, chênh 2 lần). **So với Monte Carlo:** MC cho bus_bypass 1/3000 = 3,33·10⁻⁴, Wilson ≤ 1,89·10⁻³. Seed 2 (5,28·10⁻⁴) nằm trong khoảng đó, còn seed 0 và seed 1 vượt cận trên của MC. Với k = 1 thì MC cũng rất không chắc, nên hai nguồn không mâu thuẫn rõ, nhưng cũng không xác nhận nhau tốt. `crosswalk_block` ≈ 5·10⁻⁷ khớp với MC 0/3000 (< 1,28·10⁻³), dù MC quá nhỏ để kiểm tra con số này.

**Ở đâu `top_share` > 0,5:** seed 1 `bus_bypass` (0,75), và gần ngưỡng là seed 1 `crosswalk_block` (0,50). Khi đó **một mẫu** mang 75 % ước lượng: p̂ = mean(w·y) ≈ w_max/n, tức con số thực chất là trọng số p/q của **một** kịch bản chia cho 600. Nó nói "q đã đặt quá ít khối lượng ở một vùng mà p có nhiều": chỉ một mẫu rơi vào, và nếu seed khác không rơi vào thì con số giảm hẳn. Điều này giải thích vì sao seed 1 cao nhất. Cũng nhớ rằng `top_share` thấp **không** chứng minh ước lượng đúng: nếu q bỏ sót hẳn một vùng va chạm thì không mẫu nào nặng ký và `top_share` vẫn thấp.

**Ngân sách so với Monte Carlo:** `estimate.json: budget` = **16 800** lần chạy cho cả 4 kiểu (4 × (8·300 săn + 3·600 ước lượng)), tức 4 200 lần mỗi kiểu.

- `crosswalk_block`: seed 2 có se/p = 1,34·10⁻⁷/4,75·10⁻⁷ ≈ 0,28. MC cần n ≈ 1/(p·0,28²) ≈ 1/(4,75·10⁻⁷·0,079) ≈ **2,7·10⁷ lần chạy** để đạt cùng độ chính xác, so với 4 200 của IS, tức **khoảng 6 000 lần rẻ hơn**.
- `bus_bypass`: seed 2 có se/p ≈ 0,25, MC cần ≈ 1/(5,28·10⁻⁴·0,062) ≈ 31 000 lần (khoảng 7 lần IS). Nhưng ở seed 0 se/p ≈ 0,47, MC chỉ cần ≈ 2 100 lần, tức **IS không thắng** cho một kiểu không hiếm lắm khi q lệch.

IS có lợi nhất cho sự kiện thật sự hiếm.

**Phần `other`:** lấy từ Monte Carlo C1 (`estimate.json: other` = {k: 0, n: 3000, p: 0, hi: 1,28·10⁻³, source: "mc.json"}). Nó không được săn vì `other` là "mọi thứ còn lại", không có một điểm nguy hiểm duy nhất để CEM leo lên. Vì k = 0, phần `other` cộng cận Wilson 1,28·10⁻³ vào đầu trên của tổng (`seeds[i].hi`), không cộng 0.

## 5. Các kiểu thất bại (bước C2 + 3D)

`hunt.json: found` của từng cuộc săn:

| cuộc săn | `found` | `infeasible` |
|---|---|---|
| bus_bypass | {bus_bypass: 236} | 4 |
| cut_in | {bus_bypass: 27, crosswalk_block: 1} | 2 |
| crosswalk_block | {crosswalk_block: 38, bus_bypass: 1} | 0 |
| rear_ended | {crosswalk_block: 10, bus_bypass: 2} | 49 |

**`bus_bypass`** (ví dụ B-01 trong trình xem):

![bus_bypass](evidence/shots/bus_bypass.png)

Xe buýt đỗ ở làn E2 (x ≈ 106 m). C2 chạy ở làn E2 bên cạnh, hơi phía trước xe ta, giảm tốc sau xe buýt (10,3 → 0,4 m/s) rồi chấp nhận một khe nhỏ (`cutGap` ≈ 7 m) và **lách đầu sang làn xe ta** ở x ≈ 95 m (< 112) với tốc độ < 6 m/s. Planner chỉ coi C2 là xe phía trước khi độ lệch ngang < `cutinLat` = 2,15 m. Vì thế đến t = 12,6 s xe ta mới đổi leader sang C2 (C2 ở z = 3,50, xe ta ở z = 1,75). Đường ướt (μ = 0,5, cả 4 ví dụ B-01…B-04 đều ướt) nên phanh tối đa chỉ 4,9 m/s², và xe ta quẹt hông C2 lúc t = 12,55 s.

**`crosswalk_block`** (ví dụ C-01):

![crosswalk_block](evidence/shots/crosswalk_block.png)

Xe buýt rời bến sớm (`busGo` ≈ 12,0 s) và đang lăn chậm, vẫn che vạch qua đường. Người đi bộ P1 **không ngó** qua xe buýt (`pedLook` = 0 ở cả 4 ví dụ) và bước ra lúc khoảng 14,2 s với 0,8–1,1 m/s. Xe ta tới gần với đúng tốc độ thận trọng 7,4 m/s mà không thấy P1 (`vis` = 0). Lúc t ≈ 16,3 s xe ta thấy P1 và phanh hết cỡ trên đường ướt (−4,91 m/s²), dừng ở x = 119,7 m, nên mũi xe nằm đúng mép vạch (x = 122). P1 đi tới sát thân xe thì phải đứng lại: chặn lối lúc t = 20,3 s.

**Kiểu không tìm thấy:**

- `cut_in`: cuộc săn của nó tìm 27 `bus_bypass` và 1 `crosswalk_block`, nhưng **0 `cut_in`**. Elite level kẹt ở −3,41, tức TTC tốt nhất vẫn khoảng 3,4 s.
- `rear_ended`: 0 vụ đúng kiểu, elite level −1,19.

Tôi tin ở mức **vừa phải** rằng hai kiểu này hiếm hơn `bus_bypass`: MC 0/3000 cho mỗi kiểu < 1,28·10⁻³, IS 0 hits cho `upper_zero_hits` = 2,5·10⁻². Cả hai đều là cận trên, **không** phải bằng chứng là 0. CEM có thể đơn giản là chưa tìm ra đường dẫn tới chúng.

**Nếu chỉ một cuộc săn chung:** cuộc săn `cut_in` đã cho thấy điều xảy ra. Điểm nguy hiểm kéo q về chỗ va chạm **dễ nhất**: 27/28 va chạm nó tìm được là `bus_bypass`. Còn cuộc săn `bus_bypass` tìm 236 vụ mà **không** một vụ `crosswalk_block` nào. Một cuộc săn chung sẽ co về `bus_bypass` và báo `crosswalk_block` là "không thấy". Vì thế mỗi kiểu cần một cuộc săn riêng, với điểm nguy hiểm riêng (`ped_room` cho người đi bộ).

## 6. Sửa planner (bước E + 3D)

`fix.json`:

| | policy | `fail` | `fail_hi_seen` | `fail_hi` | `unseen` | quãng đường | phanh gấp (frame/chuyến) |
|---|---|---|---|---|---|---|---|
| gốc, TRAIN (`baseline_train`) | POLICY | 3,24·10⁻⁴ | 5,33·10⁻⁴ | 5,33·10⁻⁴ | — | 214,4 m | 8,23 |
| **chọn, TRAIN** (`chosen`) | **{cutinLat: 3.0}** | 7,59·10⁻⁷ | 1,38·10⁻⁶ | 1,38·10⁻⁶ | bus_bypass | 214,4 m | 7,91 |
| **chọn, TEST** (`chosen_test`) | {cutinLat: 3.0} | **1,77·10⁻⁶** | **4,12·10⁻⁶** | **3,88·10⁻²** | bus_bypass | 214,1 m | 8,21 |
| gốc, TEST (`baseline_test`) | POLICY | **2,37·10⁻⁴** | 4,05·10⁻⁴ | **1,68·10⁻³** | — | 214,4 m | 8,48 |

TRAIN chỉ dùng để sàng: `choose_policy` chọn cận nhỏ nhất trong các ứng viên giữ ≥ 95 % quãng đường (≥ 203,7 m). Con số để báo là TEST (seed mới, cuộc săn mới nhắm vào chính policy được chọn).

**Vì sao `fail_hi` của bản sửa (3,88·10⁻²) lớn hơn cả `fail` của policy gốc (2,37·10⁻⁴) dù bản sửa tốt hơn thật:** trên TEST, cuộc săn mới nhắm vào `cutinLat` = 3,0 vẫn không ra **một** vụ `bus_bypass` nào trong 400 mẫu IS. Kiểu đó bị gắn `unseen` và cộng cận hợp lệ duy nhất là 3/(0,2·400) = **3,75·10⁻²** (`chosen_test.bus_bypass.upper_zero_hits`). Thêm 4,12·10⁻⁶ của `crosswalk_block` và 1,28·10⁻³ của các va chạm khác (0/3000) thì ra 3,88·10⁻². Đây là giới hạn của ngân sách chứ không phải của policy: với n = 400, không thể chứng nhận một kiểu đã biến mất dưới khoảng 15/n. Thứ dữ liệu **cho thấy** là `fail_hi_seen` = 4,12·10⁻⁶ so với 4,05·10⁻⁴ của gốc (khoảng 100 lần). Bản sửa gần như không mất quãng đường (214,1 so với 214,4 m) và phanh gấp ít hơn (8,21 so với 8,48 frame).

Đây đúng là cái bẫy của TRAIN mà README nhắc: ứng viên có kiểu `unseen` được `fail_hi` gần 0 trên TRAIN. Ở đây bằng chứng độc lập cho thấy cải thiện là thật: *(chạy thêm)* cả 4 kịch bản `bus_bypass` B-01…B-04 mà policy gốc va chạm đều **không** va chạm dưới `cutinLat` = 3,0. Cơ chế: C2 được coi là xe phía trước ngay khi lệch ngang < 3,0 m (tức khi nó vừa bắt đầu lách khỏi làn E2), ở B-01 là khoảng t = 10,8 s thay vì 12,6 s, tức sớm hơn gần 2 s, đủ để dừng trên đường ướt. Ngược lại, `crosswalk_block` không đổi (cùng 55 hits trên TRAIN, cùng 1,77·10⁻⁶ trên TEST). Núm này không chạm tới vấn đề người đi bộ.

![bóng mờ: bản sửa dừng sau C2, policy gốc (bóng mờ) quẹt vào C2](evidence/shots/fix_ghost.png)

**Núm "nghe thì an toàn hơn" mà làm tệ hơn:** `{T: 2.0}` (giữ khoảng cách xa hơn) có TRAIN `fail` = **3,52·10⁻³**, gấp khoảng **11 lần** gốc (3,24·10⁻⁴). `{cautionV: 6.5}` (chậm hơn gần vạch) cũng vậy: 3,50·10⁻³. *(chạy thêm, tái tạo đúng mẫu TRAIN)*: chênh lệch đến từ một kịch bản nặng ký (trọng số 0,96, `top_share` = 0,68). Đó là đường ướt, C2 rất bạo (`cutGap` = 2,4 m) đi song song xe ta.

- **Với policy gốc:** xe ta vượt qua C2 trước khi C2 kịp lách ra (lúc t = 13,5 s xe ta đã ở trước C2 3,1 m), không va chạm.
- **Với T = 2,0:** xe ta tụt lại sau khoảng 2,6 m. Trước mặt nó giờ có **một khe trống** đúng cỡ C2 chấp nhận, nên C2 bắt đầu lách ra lúc khoảng 11,5 s. Xe ta chỉ nhận C2 là leader lúc 12,5 s và đâm lúc 12,8 s.

Thế giới phản ứng lại policy: lùi lại hay chậm lại không chỉ cho xe ta thêm thời gian mà còn **mời** xe bên cạnh chen vào. Ngược lại, `{cautionV: 9.0}` (nhanh hơn) cũng tệ hơn (1,23·10⁻³), nên quan hệ giữa núm chỉnh và rủi ro không đơn điệu. Lưu ý trung thực: con số 11 lần dựa chủ yếu vào một mẫu (`top_share` 0,68), nên là tín hiệu mạnh về cơ chế nhưng yếu về độ lớn.

## 7. Từ mô phỏng ra đường thật

| Giả định (`space.py`, thị trấn B) | Dữ liệu thật thay thế, lấy từ đâu | Nếu sai thì con số sai theo hướng nào |
|---|---|---|
| **Tỉ lệ đường ướt 0,30 và μ ướt = 0,5** (cả 4 ví dụ `bus_bypass` và 4 ví dụ `crosswalk_block` đều μ = 0,5) | Số giờ mưa theo giờ trong ngày của thành phố (trạm khí tượng, ví dụ dữ liệu NOAA ISD hoặc Trung tâm Dự báo KTTV quốc gia), nhân với tỉ lệ chuyến đi theo giờ. μ đo bằng xe kéo đo trượt (skid trailer / GripTester) trên đúng loại mặt đường. | Hầu hết va chạm cần μ = 0,5. Nếu thật ra đường ướt ít hơn, hoặc μ ướt cao hơn (0,6–0,7 với nhựa tốt), thì **ta đang đánh giá quá cao** rủi ro. Nếu có mưa lớn kèm dầu, bùn (μ < 0,5) thì **đánh giá quá thấp**, và còn có cả kịch bản trên đường khô. |
| **Khe chấp nhận của xe lách xe buýt** `cutGap` (log-normal, trung vị 12 m, σ = 0,5); ví dụ va chạm có `cutGap` 2,4–10 m | Video drone ở các điểm dừng xe buýt (kiểu exiD/highD nhưng ở đô thị, tốt nhất là quay tại chỗ ở thành phố mục tiêu): đo khe dọc khi xe bắt đầu lách ra khỏi làn bị chặn. | Giao thông hỗn hợp ở Việt Nam bạo hơn nhiều, nên trung vị thật có thể dưới 12 m. Đuôi trái dày hơn thì `bus_bypass` **bị đánh giá thấp**, có thể nhiều lần, vì đây là kiểu chiếm gần hết tổng. |
| **Tỉ lệ người đi bộ ngó qua xe buýt** `pedLook` = 0,85; cả 4 ví dụ `crosswalk_block` đều `pedLook` = 0 | Đếm trên video tại vạch qua đường giữa phố, ngay sau trạm xe buýt (quan sát thực địa, hoặc tập inD / Waymo Open Motion có nhãn hướng đầu người đi bộ). | P(`crosswalk_block`) gần như tỉ lệ với (1 − pedLook). Nếu thật chỉ 60 % ngó thì (1 − 0,6)/(1 − 0,85) ≈ 2,7, tức ta **đánh giá thấp khoảng 2,7 lần**. Nếu 95 % ngó thì đánh giá cao khoảng 3 lần. |

## 8. Giới hạn

1. **Một con phố, một bố cục cố định.** Xe buýt luôn đỗ ở cùng chỗ trong làn E2 và vạch qua đường luôn ở x = 122. Không có giao lộ có đèn, rẽ trái, xe máy, đỗ xe hai bên đường. `bus_bypass` và `crosswalk_block` là đặc sản của đúng hình học này, nên P(va chạm) không phải "mỗi km" hay "mỗi chuyến" nói chung.
2. **Mọi người lái là IDM cộng luật chấp nhận khe; người đi bộ chỉ có một luật chờ.** Không có người say, mất tập trung, vượt ẩu, xe máy len lỏi, người chạy qua đường. Va chạm `other` = 0/3000 có thể chỉ vì thế giới không có các tác nhân đó.
3. **Cảm biến hoàn hảo trừ phần xe buýt che, không trễ, không đánh lái.** Xe ta biết chính xác vị trí, tốc độ mọi thứ nó nhìn thấy, phản ứng trong 1 frame, và chỉ có phanh để tránh. Mưa (30 % chuyến) không làm giảm tầm nhìn cảm biến. Ngoài ra kịch bản chỉ dài 24 s.

Nếu thêm một tuần, tôi sẽ:

- đo **độ nhạy** của tổng theo tỉ lệ đường ướt, `pedLook` và trung vị `cutGap` (chạy lại với 4 biến thể thị trấn A–D);
- chạy IS `crosswalk_block` với nhiều seed hơn và một cuộc săn khởi tạo khác để kiểm tra bẫy 3 (cuộc săn có tìm đúng cách va chạm **hay xảy ra nhất** không);
- dồn khoảng 15 000 lần chạy vào vùng `bus_bypass` dưới `cutinLat` = 3,0 để thật sự **chứng nhận** bản sửa dưới 10⁻³.

## 9. Rắc rối của bạn (bước G + chương 10)

- **Rắc rối:** một **người băng ngang giữa phố, ngoài vạch** (`hazard.json: hazards[0]`, loại `ped`, từ lề phải `side = "S"`). Người đó bước xuống lòng đường ở x = 140 m (18 m sau vạch qua đường) với 2,2 m/s (đi rất nhanh / chạy chậm) khi xe ta còn cách 28 m. Ngoài đời, đây là cảnh rất quen ở phố Việt Nam: người vừa xuống xe buýt hoặc từ quán bên đường cắt ngang thẳng sang bên kia, không đi vòng ra vạch.
- **Căn cứ:** đọc code planner (`avsim/core.py`, đoạn tính `t_enter`, `t_arrive`, `pedBuffer`) thì logic **dự đoán người đi bộ chỉ áp dụng cho người P1 ở vạch qua đường** (x = `WALK_X` = 122). Mọi người đi bộ khác chỉ được xử lý như một "xe phía trước" khi đã **đứng trong làn** của xe ta. Tôi đoán planner sẽ thấy người đó suốt mấy giây khi họ đi từ lề vào mà không làm gì. Thử nghiệm cho thấy đúng như vậy (xem dưới).
- **Giả định** (`hazard.json: space`, `exposure`, `why`):
  - `h0_x` ∈ [125, 175] m: đoạn giữa phố ngay sau vạch tới chỗ xe ta thực sự đi tới trong 24 s (người ở x lớn hơn thì xe ta không bao giờ tới gần, rắc rối không xảy ra);
  - `h0_v` ∈ [1,0; 3,0] m/s: từ đi chậm (1,07 m/s là tốc độ thiết kế đèn người đi bộ của MUTCD) qua đi nhanh tới chạy vội;
  - `h0_trig` ∈ [10, 45] m: ở 7–11 m/s là 1–4 s trước khi xe tới, cỡ khe thời gian người băng ngang chấp nhận (`pedGap` của thị trấn có trung vị 4,5 s). Dưới 10 m thì không xe nào dừng kịp, nên không nói gì về planner;
  - `exposure` = **0,02 mỗi chuyến** là **giả định** chưa có số đo: một chuyến nội đô gặp khoảng 1 người băng ngang ngoài vạch phía trước, và giả định khoảng 2 % trong số đó bước xuống khi xe còn trong 45 m. Dữ liệu để thay là video camera hành trình hoặc dữ liệu lái xe tự nhiên: đếm số lần như vậy chia cho số chuyến.
- **Cảnh dựng** (`mine.json: exact`): planner va chạm `[ego, H0, 18,4 s]`. Giao thông của cảnh lấy từ một kịch bản thị trấn B (đường khô, μ = 0,8). Trong thế giới mặc định của chương 10 thì xe dẫn L1 dừng cho người đi bộ trước nên xe ta không đâm. *(chạy thêm)* Chuỗi sự kiện ở một ví dụ tương tự: người bước xuống lúc khoảng 15,5 s, đi ngang 2,45 m/s. Xe ta (vừa qua vạch, thoát chế độ thận trọng) **tăng tốc** 7,4 → 9,4 m/s trong lúc người đi từ lề vào, vì người đó chưa phải "xe phía trước". Chỉ khi người bước vào làn (lúc 18,0 s) xe ta mới phanh hết cỡ (−7,85 m/s²) và đâm ở 18,2 s.
- **Kết quả**, `mine.json` (thị trấn B, `world` = `town`, `accepted` = **99,1 %**: 1 487/1 500, 13 bất khả thi):

| | policy gốc (`baseline`) | policy chọn {cutinLat: 3.0} (`chosen`) |
|---|---|---|
| va chạm của xe ta (`crashes`) | 91 | 88 |
| `avoidable` (tránh được bằng phanh hết cỡ ngay khi thấy) | **88** | **88** |
| `unavoidable` | 0 | 0 |
| `unattributed` | 3 | 0 |
| `low_speed` (< 1 m/s) | 1 | 1 |
| `avoidable_foreknown` | 88 | 88 |
| P(lỗi planner \| rắc rối) = `p` | **5,92·10⁻²**, Wilson [4,83·10⁻² ; 7,23·10⁻²] | 5,92·10⁻², [4,83·10⁻² ; 7,23·10⁻²] |
| mỗi chuyến = `p` × 0,02 (`per_trip`) | **1,18·10⁻³** (cận trên 1,45·10⁻³) | 1,18·10⁻³ (1,45·10⁻³) |

Đọc bảng:

- **Mọi va chạm do rắc rối gây ra đều `avoidable`.** Không vụ nào `unavoidable`, nên đây là lỗi của planner chứ không phải một tình huống vô vọng.
- **`avoidable_foreknown` = `avoidable` = 88.** Người băng ngang không bị che: xe ta thấy họ ngay từ khung hình họ bước xuống (trong `examples`, `brake_t` = `fire_t`). Che tầm nhìn không tốn gì cả. Lỗi nằm **hoàn toàn ở dự đoán**: planner thấy mà không hiểu người đó sắp đi vào làn.
- **3 vụ `unattributed`** của policy gốc là va chạm thường của thị trấn (ví dụ trong `examples`: đâm C2 lúc 12,95 s, trước khi rắc rối bắt đầu, tức một vụ `bus_bypass`). Chúng biến mất với `cutinLat` = 3,0, khớp với câu 6. Nhưng **bản sửa ở bước E không cứu được vụ nào trong 88 vụ của rắc rối này**.
- **So với lách xe buýt:** 1,18·10⁻³ mỗi chuyến (cận trên 1,45·10⁻³) cùng bậc với `bus_bypass` (`estimate.json: seeds[*].per.bus_bypass.p` = 5,3·10⁻⁴ … 3,4·10⁻³, trung vị 2,2·10⁻³). Vì `exposure` là giả định, con số mỗi chuyến chỉ đúng tới chừng nào 0,02 đúng. Sai 10 lần thì so sánh cũng sai 10 lần. Con số **có điều kiện** P(lỗi | rắc rối) ≈ 6 % thì không phụ thuộc vào giả định đó.

- **Vì sao "tránh được" là chuẩn lạc quan, theo cả hai hướng:**
  - **Đếm nhiều hơn:** người phanh trong phép thử `avoidable` biết **ngay** khung hình đầu tiên rằng người kia sẽ tiếp tục đi vào làn, và phanh tối đa không trễ. Một planner thật phải đoán ý định (nhiều người dừng lại ở mép đường) và có độ trễ. Vì vậy 88 là **cận trên** của số va chạm một planner thật có thể tránh.
  - **Đếm ít hơn:** phép thử chỉ thử **phanh**, không thử đánh lái. Một vụ `unavoidable` có thể tránh được bằng lách sang làn E2. Ở đây `unavoidable` = 0 nên hướng này không đổi con số.
- **Hình** (chương 9, mục GB-01, đúng lúc va chạm):

  ![người băng ngang giữa phố](evidence/shots/hazard.png)

- **Bạn tự lái thì sao:** *(điền sau khi thử nút "Bạn" ở chương 10)*. Thì tông chết luôn người băng qua đường :))))))
- **Cách sửa:** *(chạy thêm, 396 phiên bản cùng `space`, đếm `avoidable`; gốc = 18, quãng đường 213,6 m)*:

| núm | `avoidable` | quãng đường |
|---|---|---|
| gốc | 18 | 213,6 m |
| `pedBuffer` 6,0 | 18 | 213,6 m |
| `cutinLat` 3,0 | 18 | 213,3 m |
| `T` 2,5 | 17 | 189,9 m |
| `b` 4,0 | 20 | 212,8 m |
| `v0` 10 | 14 | 184,4 m |
| `v0` 8 | 1 | 167,1 m |
| `cautionV` 5 + `cautionRange` 90 | 5 | 166,5 m |

  **Không núm nào sửa được mà giữ ≥ 95 % quãng đường.** `pedBuffer` không có tác dụng vì nó chỉ áp dụng cho người ở vạch. Chỉ chạy chậm hẳn (`v0` = 8) mới giảm được, nhưng mất 22 % quãng đường, và một phần mức giảm chỉ vì xe ta không đi tới chỗ người băng ngang. Planner cần **logic mới**: áp dụng phép dự đoán đang dùng cho P1 cho **mọi người đi bộ nhìn thấy**, không chỉ ở x = 122:
  - với mỗi người đang có vận tốc ngang hướng vào làn xe ta, tính t_enter = khoảng cách tới mép làn / vận tốc ngang và t_arrive = (x_người − x_xe) / v_xe;
  - nếu t_enter < t_arrive + `pedBuffer`, đặt một "vạch dừng ảo" trước x của người đó (như `gp` đang làm cho vạch qua đường);
  - đồng thời, khi có người đang ở trên lòng đường phía trước, giới hạn tốc độ sao cho quãng phanh v²/(2μg) nhỏ hơn khoảng cách tới họ, và không tăng tốc trong lúc đó.

  Trong cảnh này, thêm logic đó thì xe ta bắt đầu giảm tốc từ lúc người bước xuống (khoảng 2,5 s trước va chạm), đúng như phép thử `avoidable` cho thấy là đủ để dừng.
