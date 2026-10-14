MSSV: 202602903

# Lab 11 · Scenario Forge — báo cáo

Thị trấn: **A** (`meta.json: _run.town`). Ngân sách chuẩn, không dùng `--quick`. Seed từ MSSV, giữ nguyên simulator, phân phối và danh sách ứng viên gốc.

## 1. Phát lại log có trung thực không?

`replay.json`: n=400, dropped_human_crash_logs=0, n_used=400, crash_in_both=0.

| Cách chạy | Tổng thất bại | front | rear | side |
|---|---:|---:|---:|---:|
| replay | 108 | 0 | 99 | 9 |
| reactive | 0 | 0 | 0 | 0 |

Ví dụ `replay.json: replay_only`, scenario=5, contact=rear, min_ttc_reactive=2.518433 s. Trong log, xe bám đuôi tiếp tục đi theo quỹ đạo ghi khi người lái điều khiển; planner phanh sớm hơn nhưng log không biết để phanh theo. Xe bám đuôi tông xe ta trong replay; trong thế giới phản ứng nó điều chỉnh theo xe ta và không va chạm. Vì vậy đây là va chạm giả do phát lại, không chứng minh planner nguy hiểm ở cùng tình huống phản ứng.

## 2. Kịch bản bất khả thi

| Luật trong `is_feasible` | Căn cứ |
|---|---|
| `(π/cutDur)² × 3.5/2 ≤ μg` | Gia tốc ngang khi chuyển làn cosine không vượt lực bám lốp |
| `cutBrake ≤ μg` | Phanh không vượt lực ma sát mặt đường |
| `pedV ≤ 2.5 m/s` | Miền đi bộ của lab, loại người chạy khỏi phân phối đang mô hình hóa |
| `0 < cutV ≤ 22 m/s` | Miền xe tiến về phía trước và trần vận tốc của thế giới lab |

Hai ngưỡng tốc độ là giới hạn mô hình, không phải khẳng định mọi người/xe thật đều không thể vượt chúng. `cutDur ≤ 0` bị loại vì thời gian chuyển làn không hợp lệ.

| Cuộc săn (`hunt.json`) | infeasible | found |
|---|---:|---|
| bus_bypass | 0 | `{"bus_bypass": 196}` |
| cut_in | 5 | `{"bus_bypass": 12}` |
| crosswalk_block | 7 | `{"bus_bypass": 2, "crosswalk_block": 71}` |
| rear_ended | 2 | `{"bus_bypass": 4, "crosswalk_block": 4}` |

Cuộc săn sinh nhiều kịch bản bất khả thi nhất là `crosswalk_block` (7). CEM ưu tiên nguy hiểm nên dễ kéo thời gian chuyển làn xuống, tốc độ lên hoặc phanh vượt lực bám; sàng feasibility trước khi mô phỏng ngăn chúng đóng góp thất bại. Nếu đếm các mẫu đó là va chạm, ước lượng sẽ bị đẩy cao bởi các cảnh nằm ngoài miền vật lý/mô hình.

## 3. Monte Carlo cho bạn biết gì

`mc.json`: crashes=5, n=3000, p=0.0016666667; Wilson 95% [0.00071209344, 0.0038958726]. Với xấp xỉ nhị thức, để nửa độ rộng là 20% của p: n≈1.96²(1−p)/(0.2²p)≈96/p≈57,600 lượt. Đây là dự báo từ p̂ rất nhiễu, không phải chứng nhận chính xác.

| Kiểu (`mc.json: per_mode`) | k | Wilson lo | Wilson hi |
|---|---:|---:|---:|
| bus_bypass | 5 | 0.00071209344 | 0.0038958726 |
| cut_in | 0 | 0 | 0.0012788957 |
| crosswalk_block | 0 | 0 | 0.0012788957 |
| rear_ended | 0 | 0 | 0.0012788957 |
| other | 0 | 0 | 0.0012788957 |

Các kiểu k=0 được báo bằng cận trên Wilson trong bảng; không kết luận xác suất thật bằng 0.

## 4. Importance sampling, ESS và độ suy biến

Tất cả số sau từ `estimate.json: seeds[].per`. Với hits=0, p̂=0 chỉ là thống kê mẫu; cột cận 0-hit là giới hạn cần báo về xác suất thật.

| seed | kiểu | p̂ | hits | ess | top_share | upper_zero_hits |
|---:|---|---:|---:|---:|---:|---:|
| 0 | bus_bypass | 0.00023323167 | 37 | 120.12056 | 0.19837702 | — |
| 0 | cut_in | 0 | 0 | 114.82765 | 0 | 0.025 |
| 0 | crosswalk_block | 1.9994011e-07 | 90 | 126.0972 | 0.23937647 | — |
| 0 | rear_ended | 0 | 0 | 126.39641 | 0 | 0.025 |
| 1 | bus_bypass | 0.00091243411 | 48 | 127.40909 | 0.20654243 | — |
| 1 | cut_in | 0 | 0 | 116.88095 | 0 | 0.025 |
| 1 | crosswalk_block | 9.4806393e-07 | 78 | 136.08348 | 0.48946802 | — |
| 1 | rear_ended | 0 | 0 | 109.20835 | 0 | 0.025 |
| 2 | bus_bypass | 0.00059394912 | 39 | 134.12723 | 0.31311203 | — |
| 2 | cut_in | 0 | 0 | 107.85307 | 0 | 0.025 |
| 2 | crosswalk_block | 9.0254296e-07 | 100 | 142.03799 | 0.68176875 | — |
| 2 | rear_ended | 0 | 0 | 135.03099 | 0 | 0.025 |

| seed | total | lo | hi |
|---:|---:|---:|---:|
| 0 | 0.00023343161 | 0.00010375844 | 0.0016420004 |
| 1 | 0.00091338218 | 0.00037859354 | 0.0027270665 |
| 2 | 0.00059485166 | 0.00016768631 | 0.0023009127 |

Trung vị total=0.00059485166; min–max=0.00023343161–0.00091338218, so với MC=0.0016666667. Các seed chênh trên 2 lần, nên chưa ổn định.

MC p̂ / trung vị IS = 2.801819; nếu lệch nhiều, đó là dấu hiệu q có thể bỏ sót vùng va chạm hoặc ước lượng còn nhiễu, không được chọn số đẹp hơn.

`top_share>0.5`: crosswalk_block/seed 2 (0.68176875). top_share là phần tổng w·y do mẫu lớn nhất đóng góp; >0.5 nghĩa một mẫu quyết định hơn nửa ước lượng. ESS đo độ lệch trọng số, không phải số lần thử độc lập và không phải bằng chứng q đã thấy mọi vùng va chạm.

`estimate.json: budget`=16,800 lượt (4×(8×300+3×600)); C1 thêm 3,000 lượt. MC ở độ chính xác tương đối ±20% cần khoảng 57,600 lượt từ p̂ C1; nếu lấy trung vị IS để dự trù, cần khoảng 161,385 lượt. IS tiết kiệm ở vùng hiếm nhưng độ chính xác thực ở ngân sách này chưa được bảo đảm; CI ±1.96·se chỉ xấp xỉ, dễ hẹp sai khi trọng số suy biến.

`estimate.json: other`={"k": 0, "n": 3000, "p": 0.0, "hi": 0.0012788956648046956, "source": "mc.json"}. Phần other lấy từ MC vì không có điểm nguy hiểm/đề xuất chuyên biệt để săn. Khi MC không thấy other, tổng vẫn cộng cận Wilson của nó vào hi. Với IS không hit, báo 3/(0.2×600)=0.025; không dùng 3/ESS.

Trong estimate.json, hi của total chưa cộng các upper_zero_hits cho cut_in/rear_ended. Giữ nguyên số phát ra nhưng không coi khoảng đó là chứng nhận 95% cho toàn bộ thất bại khi còn kiểu chưa thấy; cận từng kiểu nằm trong bảng trên.

## 5. Các kiểu thất bại

### bus_bypass

![bus_bypass, t=12.6 s](evidence/shots/05-bus_bypass.png)

Xe C2 tiến tới xe buýt đỗ trong E2 rồi lấn sang E1 để lách; ego vẫn tiến và chạm C2 đang chạy chậm trước x=112 m. Khe nhập làn của C2 và thời điểm ego tới gần tương tác, nên một TTC nhỏ ở C2 có thể kéo cuộc săn về kiểu này. Trong cảnh này, x_C2=94.6897 m, v_C2=1.7322 m/s ở va chạm; P1 mới bắt đầu bước lúc 12.55 s nên tiếp xúc là với C2, không phải người đi bộ. Kịch bản `B-01` trong `viewer.json`, sự kiện `contact` ở t=12.6 s.

### crosswalk_block

![crosswalk_block, t=20.9 s](evidence/shots/05-crosswalk_block.png)

P1 chờ khe đủ lớn theo xe nó nhìn thấy rồi bước từ sau xe buýt. Ego nhìn thấy người muộn và phanh, nhưng thân xe dừng trên vạch nên P1 phải đứng chờ. Đây là chặn lối, không nhất thiết là tiếp xúc thân người. P1 bắt đầu bước lúc 14.15 s; đường ướt mu=0.5, pedLook=0; khi block xảy ra ego ở x=119.7329 m và v=0.0000 m/s. BusGo=12.0041 s là thời điểm đủ điều kiện xe buýt rời trạm, không bảo đảm nó đã rời trạm ngay. Kịch bản `C-01` trong `viewer.json`, sự kiện `block` ở t=20.9 s.

Chưa tìm thấy: cut_in, rear_ended. Không tìm thấy với ngân sách 8×300 không chứng minh xác suất bằng 0; các cận MC/IS ở câu 3–4 thể hiện mức bất định. Bảng found ở câu 2 cho thấy các cuộc săn có thể tìm nhiều va chạm của cùng kiểu ngoài mục tiêu; một điểm nguy hiểm chung dễ co về vùng dễ tối ưu và bỏ qua kiểu khác. Vì vậy cần săn riêng và vẫn giữ hỗn hợp phòng thủ.

## 6. Sửa planner

`fix.json: chosen.policy` = `{"cutinLat": 3.0}`. Chọn duy nhất theo TRAIN: fail_hi nhỏ nhất, giữ ít nhất 95% progress baseline; nếu hòa, chọn progress lớn hơn. Không chọn lại theo TEST. TEST dùng seed mới, tìm kiếm mới nhắm vào từng policy và 3.000 mẫu tự nhiên mới.

| Nguồn trong fix.json | fail | fail_hi_seen | fail_hi | progress (m) | harsh (khung/lượt) |
|---|---:|---:|---:|---:|---:|
| Gốc TRAIN (baseline_train) | 0.001065455 | 0.0018336306 | 0.0018336306 | 213.96818 | 8.4492188 |
| Chọn TRAIN (chosen) | 3.7236909e-07 | 6.8878675e-07 | 6.8878675e-07 | 213.94512 | 8.15625 |
| Gốc TEST (baseline_test) | 0.00055438948 | 0.00088472703 | 0.0021636227 | 214.22039 | 8.611 |
| Chọn TEST (chosen_test) | 7.4853487e-07 | 1.3666967e-06 | 0.038780262 | 214.12071 | 8.0766667 |

TRAIN: mất quãng đường 0.0108%; TEST: 0.0465%. `harsh` đếm khung có a<−3 m/s², không phải số đợt phanh độc lập. Quãng đường trong đánh giá được cắt ở va chạm.

TEST chosen.unseen=['bus_bypass']. fail_hi cộng 3/(0.2×400)=0.0375 cho mỗi kiểu không hit, cùng cận Wilson của va chạm ngoài hai kiểu được săn. Vì vậy fail_hi có thể lớn hơn fail của baseline: một bên là cận có bất định, một bên là ước lượng điểm. fail_hi_seen chỉ cộng kiểu đã thấy, không phải cận hợp lệ cho toàn policy.

![Bản sửa và bóng policy gốc](evidence/shots/06-fix-ghost.png)

Cùng params của `B-01` trong viewer.json, policy gốc có contact với C2 ở 12.6 s; policy được chọn không có sự kiện ego trong 24 s. Ảnh dùng bản sửa làm xe chính và policy gốc làm bóng mờ; `evidence/fix_comparison.json` chỉ đổi cách xem, không đổi tham số kịch bản hay policy.

Các số fail=0 hoặc thành phần p̂=0 là ước lượng mẫu trên TRAIN, không chứng nhận xác suất thật bằng 0; kiểm tra toàn policy dùng TEST và cận cho unseen.

| Ứng viên TRAIN | fail | fail_hi | progress |
|---|---:|---:|---:|
| `{"cautionV": 5.0}` | 0.00048643261 | 0.0013310328 | 172.54073 |
| `{"cautionV": 6.5}` | 0.00056579675 | 0.0011969682 | 197.32328 |
| `{"cautionV": 6.0}` | 0.00062277625 | 0.001130762 | 189.72684 |
| `{"cautionV": 9.0}` | 0.0026056973 | 0.0056737717 | 226.50671 |
| `{"cautionV": 8.5}` | 0.002333251 | 0.0053884821 | 222.78174 |
| `{"cutinLat": 2.6}` | 3.0529279e-05 | 8.1866186e-05 | 213.95573 |
| `{"cutinLat": 3.0}` | 3.7236909e-07 | 6.8878675e-07 | 213.94512 |
| `{"cutinLat": 3.3}` | 3.7236909e-07 | 6.8878675e-07 | 213.93441 |
| `{"T": 2.0}` | 0.00067267451 | 0.0013176618 | 203.71865 |
| `{"cutinLat": 3.0, "cautionV": 6.5}` | 1.115993e-06 | 2.6502806e-06 | 197.26668 |
| `{"cutinLat": 3.0, "cautionV": 7.0}` | 1.1553422e-06 | 2.690282e-06 | 206.38758 |
| `{"cutinLat": 2.6, "cautionV": 7.0, "pedBuffer": 3.5}` | 6.1823063e-05 | 0.00017193671 | 206.65949 |

Ví dụ phản tác dụng, cùng cutinLat=3.0: giảm cautionV từ mặc định 7.5 xuống 7.0 m/s tăng TRAIN fail từ 3.7236909e-07 lên 1.1553422e-06 (khoảng 3.103 lần); crosswalk_block hits từ 71 lên 75. Khoảng xấp xỉ còn chồng nhau và top_share lớn, nên chưa kết luận mức tăng này ổn định ngoài mẫu.

Đối chứng phụ `policy_counterexample.json` dùng đúng các mẫu TRAIN của fix (seed search+11, rút bus_bypass trước rồi crosswalk_block), không dùng TEST để chọn. Ở scenario_index=11, cautionV=7.5 không có sự kiện ego; cautionV=7.0 có block với P1 ở 20.0 s. P1 bắt đầu ở 14.55 s / 14.55 s tương ứng. Ego ở x=106.5845 / 104.7026 m khi P1 bắt đầu. P1 được thấy (sau lúc bước) ở 15.55 / 16.0 s; cuối lượt P1 ở z=-0.7030 / 3.4887 m. Ở cảnh đối chứng này, giảm cautionV làm P1 hiện ra sau che khuất muộn hơn 0.45 s dù tốc độ thấp hơn; ego bắt đầu thấy/phanh khi đã tới x=114.7608 m thay vì 113.9600 m. Nó dừng ở x=119.5228 m thay vì 119.3592 m, đủ để thân xe chặn đường P1. P1 phản ứng bằng đứng lại (z=3.4887 m), trong khi ở policy nhanh hơn P1 tiếp tục qua (z=−0.7030 m). Vì vậy thế giới phản ứng và hình học che khuất có thể đảo chiều trực giác “chậm hơn an toàn hơn”. Đây là một cảnh kiểm chứng tác dụng phụ cụ thể, không chứng minh một tỷ lệ nguy hiểm chung.

## 7. Từ mô phỏng ra đường thật

| Giả định đang dùng (space.py, thị trấn A) | Dữ liệu cần thu và nguồn cụ thể | Hướng sai lệch nếu giả định sai |
|---|---|---|
| cutGap lognormal, median=12 m | Video drone/camera cố định tại điểm chuyển làn và trạm xe buýt; gán nhãn khe thực lúc người lái bắt đầu nhập làn, phân tầng theo tốc độ và thời tiết | Nếu lái xe thật chấp nhận khe nhỏ hơn, mô hình có thể đánh giá thấp cut-in/bus-bypass; ảnh hưởng còn phụ thuộc phản ứng ego |
| pedLook Bernoulli=0.85 | Video cận cảnh tại trạm xe buýt/vạch qua đường, gán nhãn có ngó qua đầu xe buýt trước khi bước hay không; đo đồng thời khe và thời điểm bước | Nếu ít người ngó hơn 85%, mô phỏng có thể đánh giá thấp crosswalk-block; cần ước lượng cả tương quan với khe |
| wet=0.20, μ ướt=0.5/khô=0.8 | Log mưa theo chuyến từ đội xe, cảm biến mặt đường và thử phanh trên sân thử với từng loại lốp/tốc độ | Nếu đường ướt thường hơn hoặc μ thấp hơn, quãng phanh dài hơn và thất bại thường tăng; mô hình hiện có thể đánh giá thấp rủi ro |

Đây là kế hoạch lấy dữ liệu, chưa tuyên bố đã đo hoặc dùng một bộ dữ liệu thật nào. Cần ước lượng tương quan, không chỉ thay trung bình từng biến độc lập.

## 8. Giới hạn

1. Chỉ một đoạn phố, 24 giây, số tác nhân cố định: xác suất không đại diện hành trình dài, nhiều giao lộ hay mọi loại đường.
2. Xe khác chủ yếu IDM và quy tắc khe; người đi bộ dùng ngưỡng chờ: không đại diện mất tập trung, phản ứng chậm, thương lượng hoặc người cố tình chen.
3. Cảm biến/hình học gần lý tưởng ngoài che khuất: chưa đo sai phát hiện, độ trễ, lỗi tracking và dự đoán; cũng không đánh giá mức độ chấn thương khi có tiếp xúc.

Nếu có thêm một tuần: thu/hiệu chỉnh khe nhập làn và hành vi ngó đầu xe buýt; chạy nhiều seed và ngân sách lớn hơn trên holdout; thêm trễ cảm biến/phản ứng, rồi báo đường cong độ ổn định CI/ESS/top_share theo ngân sách.

## 9. Rắc rối của bạn

Xe đạp đang ở E2 sát mép trái, lạng sang E1 khi ego tới gần (ví dụ tránh vật cản). Planner có thể nhận vật thể phía trước quá muộn vì nó chưa ở trong hành lang đi theo; tăng cutinLat cho ô tô không nhất thiết sửa dự đoán chuyển động ngang của xe đạp.

`hazard.json: why` ghi đầy đủ căn cứ hình học/vật lý cho miền thử: x=−10–40 m (30–80 m trước vị trí đầu ego −40 m), v=3–5 m/s, trig=12–28 m, swerve=1.8–2.3 m, swDur=2–2.6 s. Gia tốc ngang lớn nhất π²×2.3/(2×2²)=2.8375 m/s² <3 m/s². Đây là miền thiết kế kiểm thử, không phải khoảng rút từ thống kê thực địa.

**Phạm vi:** `hazard.json: world=scene`, exposure=1 vì mỗi lượt thử cố ý đặt một xe đạp. `mine.json: conditional_on` là đúng cảnh mặc định, không phải hỗn hợp thị trấn A. Exposure này là mức phơi nhiễm của thí nghiệm, không phải một tần suất ngoài đời; không nhân để so tần suất chuyến với bus_bypass. Muốn so cần đo số lần xe đạp lạng trên mỗi chuyến thật và chạy world=town. Chọn phạm vi có điều kiện này tránh gán một tỷ lệ thực tế không có dữ liệu.

`mine.json: exact` = `{"planner": [["ego", "H0", 4.0], ["C2", "H0", 6.2], ["C2", "P1", 14.25]]}`.

`mine.json: accepted`=1 (1500/1500 khả thi); `infeasible`=0.

| policy (mine.json) | n | crashes | avoidable | unavoidable | unattributed | low_speed | avoidable_foreknown | p | Wilson lo | Wilson hi |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| baseline | 1500 | 850 | 17 | 13 | 820 | 19 | 17 | 0.011333333 | 0.0070878803 | 0.018075408 |
| chosen | 1500 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0.0025545244 |

avoidable là không va chạm nếu cùng planner phanh tối đa ngay khung đầu hazard đã hoạt động **và nhìn thấy**; unavoidable vẫn va chạm theo thử phanh này; unattributed là va chạm không quy được cho hazard. low_speed (<1 m/s) là cờ chồng lên các nhóm, không cộng vào tổng lần nữa. avoidable_foreknown cho phanh từ lúc hazard bắt đầu dù còn bị che, nên chênh với avoidable cho biết giá của che khuất.

Chuẩn phanh là lạc quan vì biết hazard ngay khi nhìn thấy, không có trễ phản ứng và phanh hết lực: có thể đếm nhiều hơn số lỗi một planner triển khai thực tránh được. Nhưng chỉ thử phanh, bỏ qua đánh lái, có thể đếm ít hơn số va chạm có thể tránh bằng điều khiển khác. unavoidable ở đây không đồng nghĩa bất khả tránh bằng mọi hành động.

![Xe đạp, chương 10, va chạm 4.0 s](evidence/shots/09-bike-studio.png)

So sánh: avoidable baseline=17/1500, chosen=0/1500. Bản sửa ở E không thấy lỗi tránh được trong mẫu xe đạp này; vẫn cần cận Wilson để đánh giá. Tần suất bus_bypass thị trấn trong ba seed IS là 0.00023323167, 0.00091243411, 0.00059394912; đó là xác suất vô điều kiện của mô hình thị trấn, không so trực tiếp với p có điều kiện cảnh xe đạp.

Chưa có thử nghiệm người tự lái/egoAcc; không tuyên bố người tránh được. Oracle phanh là mô phỏng, không phải lượt tự lái của sinh viên.

Đề xuất sửa: dự báo quỹ đạo ngang và vùng có thể chiếm của xe đạp trong 2–3 giây, coi xe đạp đang tiến vào E1 là vật cản trước khi tâm nó nằm trong hành lang hiện tại; giảm tốc theo TTC/quãng phanh tới vùng giao nhau. Dùng ngưỡng riêng theo kích thước xe đạp và độ bất định, kiểm tra lại tông đuôi, quãng đường trên TRAIN rồi TEST mới. Chỉnh cutinLat có thể là sàng lọc ban đầu; cần logic theo vận tốc ngang để tránh phanh vô cớ với xe đạp đi thẳng.
