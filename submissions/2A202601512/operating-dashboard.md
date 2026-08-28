# Operating Dashboard — Finora Plus

- Học viên: Trần Văn Ngọc
- Mã học viên: 2A202601512
- Mô hình: B2C
- Cập nhật: 2026-08-28
- North Star: D30→D60 paid-cohort retention mục tiêu ≥97,2%

## Chẩn đoán mô hình

Chúng tôi là B2C vì tiền đến từ người dùng Finora Plus trả từ ngân sách phần mềm cá nhân hoặc financial wellness, người dùng thật là người dùng ví Finora quản lý chi tiêu và mục tiêu tiết kiệm, và chúng tôi có bề mặt trực tiếp với người dùng cuối qua chatbot Finora trong app từ Dashboard hoặc transaction đến Hỏi Finora.

| Dữ liệu đầu vào | Trạng thái | Nằm ở đâu hoặc cần gì để đo | Ngày có số |
|---|---|---|---|
| Unit economics Day 24 | TRONG 2 TUẦN | Re-run mô hình với ARPU $4,80, CAC $42,24, Cost/Job $0,1067 và cohort retention của Finora; không dùng ARPU VND cũ. | 2026-09-11 |
| Value Metric và Cost/Job Day 25 | ĐO ĐƯỢC | `TranVanNgoc_Day25_onepager.docx`: 12 eligible job/tháng, giá $0,40/job, ARPU $4,80, Cost/Job model $0,1067. | 2026-08-28 |

## Kiểm kê đèn ứng viên

| Đèn ứng viên từ handbook | Tầng | Trạng thái | Bằng chứng hiện có hoặc kế hoạch đo |
|---|---|---|---|
| Đường cong retention có phẳng không | L | 🔧 | Tạo paid cohort D0, D30 và D60 từ subscription export; có số 2026-11-26. |
| Activation rate | L | 🔧 | Log `chatbot_entry_seen` và `eligible_job_completed` trong pilot; baseline 2026-09-11. |
| p95 cost/user/tháng ÷ ARPU | L | 🔧 | Log token, model và AI cost theo paid user; p95 khả dụng khi có 100 paid user. |
| Trial → paid | O | ❌ | Chưa chốt hard paywall hay freemium; chỉ thêm khi cấu hình paywall được chọn. |
| Retention M12 | O | ❌ | Chưa có thời gian quan sát 12 tháng; thay bằng D30→D60 cohort retention. |
| Chi phí free tier ÷ tổng COGS | O | ❌ | Chưa chốt free tier; không đưa vào dashboard tuần khi chưa có chính sách quota. |
| Tỷ lệ refund | O | ❌ | Chưa có paid transaction; lấy từ billing export sau khi mở paid cohort. |
| LTV/CAC · CAC payback · Gross margin | G | 🔧 | Dùng Day 25 để tính model hiện tại, rồi thay bằng ledger revenue và AI cost thực tế. |

## Đèn báo sớm

| ID | Đèn | Định nghĩa và công thức | Nhịp · Owner | Hiện tại | 🟢 | 🟡 | 🔴 | Nguồn | Ngày kiểm tra | Báo trước cho | Luật |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| L-01 | D30→D60 paid-cohort retention | Số user paid active ở D60 ÷ số user paid active ở D30; paid active là subscription chưa hết hạn trong cửa sổ 7 ngày. | TUẦN · PRODUCT OPS | Chưa có cohort | ≥97,2% | ≥95% và <97,2% | <95% | [MH] MH-02: retention tối thiểu để LTV/CAC đạt 3×. | 2026-11-26 | G-02 LTV/CAC sau 1–3 tháng | R-01 |
| L-02 | Activation 24h | User mới hoàn tất ít nhất 1 eligible job trong 24 giờ sau `chatbot_entry_seen` ÷ user mới có `chatbot_entry_seen`; eligible job loại fallback và tác vụ tiền thật cần xác nhận. | NGÀY · PRODUCT OPS | Chưa có baseline | >130% baseline | ≥70% và ≤130% baseline | <70% baseline | [TB] Chốt baseline pilot từ event log trong 2 tuần. | 2026-09-11 | L-01 retention trong 2–4 tuần | R-02 |
| L-03 | p95 AI cost/user ÷ ARPU | Phân vị 95 của tổng token, inference và tool cost trên mỗi paid user rolling 28 ngày ÷ ARPU rolling 28 ngày. | TUẦN · FINOPS | Chưa có p95; mean model 26,7% | <30% | ≥30% và ≤60% | >60% | [MH] MH-01: guardrail cost theo giá $0,40/job và GM. | 2026-09-11 | G-01 Gross Margin trong kỳ billing | R-03 |
| L-04 | Confirmed plan value rate | Eligible job có event `plan_confirmed_helpful` trong 24 giờ ÷ eligible job đã hiện feedback prompt; event chỉ ghi sau khi user tự chọn hữu ích hoặc đã áp dụng. | TUẦN · PRODUCT OPS | Chưa có baseline | >130% baseline | ≥70% và ≤130% baseline | <70% baseline | [TB] Thêm event feedback và chốt baseline pilot 2 tuần. | 2026-09-11 | L-01 retention và G-02 LTV/CAC | R-05 |

## Đèn vận hành

| ID | Đèn | Định nghĩa và công thức | Nhịp · Owner | Hiện tại | 🟢 | 🟡 | 🔴 | Nguồn | Ngày kiểm tra | Báo trước cho | Luật |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| O-01 | Containment | Eligible request hoàn tất không fallback hoặc human handoff ÷ tổng eligible request; request có fallback tính là không containment. | TUẦN · PRODUCT OPS | Chưa đo thực tế | ≥70% | ≥62,2% và <70% | <62,2% | [MH] MH-03: mục tiêu và breakeven containment từ Day 25. | 2026-09-11 | O-02 Cost/Job và G-01 Gross Margin | R-04 |
| O-02 | Chi phí AI trên mỗi job | Tổng token, inference, tool cost và fallback cost liên quan ÷ eligible completed jobs trong tuần. | TUẦN · FINOPS | $0,1067/job model | ≤$0,12 | >$0,12 và ≤$0,20 | >$0,20 | [MH] MH-01: suy từ giá $0,40/job và GM 70% hoặc 50%. | 2026-09-11 | G-01 Gross Margin tháng | R-03 |

## Đèn kết quả

| ID | Đèn | Định nghĩa và công thức | Nhịp · Owner | Hiện tại | 🟢 | 🟡 | 🔴 | Nguồn | Ngày kiểm tra | Báo trước cho | Luật |
|---|---|---|---|---:|---|---|---|---|---|---|---|
| G-01 | Gross Margin | Subscription revenue trừ toàn bộ COGS gồm AI cost, tool cost và allocated infrastructure, sau đó chia subscription revenue. | THÁNG · FINANCE | 73,3% model | ≥70% | ≥50% và <70% | <50% | [MH] MH-01: Day 25 chỉ test Plus khi GM đạt 70%; dưới 50% là cấu trúc gãy. | 2026-09-27 | Metric kết quả, không dự báo tầng dưới | R-03 |
| G-02 | LTV/CAC | Monthly gross profit nhân với 1 ÷ monthly churn, sau đó chia CAC; chỉ dùng cohort churn thực tế sau khi có D60. | THÁNG · FOUNDER | 2,78× model với churn 3% | ≥3× | ≥1× và <3× | <1× | [MH] MH-02: Day 24 healthy tại 3×; model Finora hiện chưa xanh. | 2026-11-26 | Metric kết quả, không dự báo tầng dưới | R-01 |

## Luật quyết định

| ID | NẾU | TRONG | VÀ | THÌ | KHÔNG THÌ | Luật dừng? |
|---|---|---|---|---|---|---|
| R-01 | L-01 D30→D60 paid-cohort retention <95% | 2 cohort paid liên tiếp | Mỗi cohort có ≥30 user và subscription export đủ 100% record | Đóng băng toàn bộ paid acquisition 3 tuần và chuyển sprint sang luồng chatbot entry đến first eligible job. | Không tăng ngân sách ads để bù churn. | CÓ |
| R-02 | L-02 Activation 24h <70% baseline | 2 cohort tuần liên tiếp | Mỗi cohort có ≥30 user thấy chatbot entry và baseline đã chốt 2026-09-11 | Dừng thử paywall mới và chuyển onboarding sang deep link Dashboard hoặc transaction với một CTA Hỏi Finora. | Không giảm giá Plus để cứu activation. | CÓ |
| R-03 | L-03 p95 AI cost/user ÷ ARPU >60% | 4 snapshot tuần rolling 28 ngày liên tiếp | Có ≥100 paid user và ≥95% event có token, model và AI cost | Giới hạn quota nhóm p95 và chuyển request vượt quota sang model rẻ hơn hoặc gói credit riêng. | Không tăng giá đại trà cho toàn bộ user. | KHÔNG |
| R-04 | O-01 Containment <62,2% | 2 tuần liên tiếp | Có ≥120 eligible request và fallback log có reason code | Dừng mở rộng beta và chuyển hai nhóm misroute lớn nhất sang fallback hoặc HITL cho đến khi containment phục hồi. | Không mở acquisition campaign khi bot chưa tự xử lý request hiện hữu. | CÓ |
| R-05 | L-04 Confirmed plan value rate <70% baseline | 2 tuần liên tiếp | Có ≥120 eligible job và feedback prompt response rate ≥70% | Dừng mở rộng financial intent ngoài 3 job lõi và chuyển backlog sang sửa ba intent có confirmed value thấp nhất. | Không thêm tính năng mới để che vấn đề giá trị. | CÓ |

## Cổng gác 90 ngày

| Ngày | Metric gác cổng | Ngưỡng | Bằng chứng vật lý | Nếu đạt | Nếu trượt |
|---:|---|---|---|---|---|
| 30 | Event instrumentation completeness = eligible request có đủ request_started, response_shown, fallback, user_feedback, token và model cost ÷ tổng eligible request | ≥95%, với ≥120 eligible request | `pilot-event-export.csv` và `pilot-instrumentation-audit.md` | GO | FIX |
| 60 | O-01 Containment | ≥70%, với ≥360 eligible request | `weekly-containment-report.md` và raw fallback log có reason code | GO | FIX |
| 90 | G-01 Gross Margin thực tế | ≥70%, với ≥30 paid user và ≥360 paid eligible job | billing export, AI-cost ledger và `unit-economics-close.md` | GO | PIVOT |

## Kill criteria

KILL hướng Finora Plus subscription $4,80/tháng nếu đến 2026-12-26, sau 2 sprint routing hoặc quota và ít nhất 360 paid eligible job, Gross Margin thực tế vẫn <50% trong 2 kỳ chốt 30 ngày liên tiếp; dừng acquisition, dừng mở rộng và dừng đầu tư vào packaging hiện tại.

## Chưa đo được

| Đèn hoặc giả định | Cần gì để đo | Ai chịu trách nhiệm | Ngày có số |
|---|---|---|---|
| p95 AI cost/user và token breakdown | Event log chứa user_id, paid status, model, input token, output token, tool cost và timestamp; export rolling 28 ngày. | FINOPS | 2026-09-11 |
| D30→D60 paid-cohort retention | Subscription status export theo cohort D0, D30 và D60; định nghĩa paid active dùng cửa sổ 7 ngày. | PRODUCT OPS | 2026-11-26 |

## Phụ lục ngưỡng suy từ mô hình

| ID | Metric | Input Day 24–25 | Phép tính | Kết quả và ngưỡng áp dụng |
|---|---|---|---|---|
| MH-01 | Cost/Job và Gross Margin | Giá $0,40/job; GM xanh 70%; GM đỏ 50%; Cost/Job model $0,1067/job. | Cost xanh = $0,40 × (1 − 70%) = $0,12/job; cost đỏ = $0,40 × (1 − 50%) = $0,20/job. | Áp dụng L-03, O-02, G-01: xanh ≤$0,12; đỏ >$0,20; model hiện ở $0,1067. |
| MH-02 | Retention và LTV/CAC | 12 job/tháng; Cost/Job $0,1067; ARPU $4,80/tháng; CAC $42,24; mục tiêu LTV/CAC 3×. | COGS = 12 × $0,1067 = $1,2804; GP = $4,80 − $1,2804 = $3,5196; lifetime = (3 × $42,24) ÷ $3,5196 = 36 tháng; retention = 1 − 1÷36 = 97,2%. | Áp dụng L-01 và G-02: retention xanh ≥97,2%; churn Day 24 3% cho LTV/CAC 2,78×. |
| MH-03 | Containment | Day 25 target containment 70%; breakeven containment 62,2%. | Cushion vận hành = 70% − 62,2% = 7,8 điểm phần trăm. | Áp dụng O-01: xanh ≥70%; vàng 62,2% đến dưới 70%; đỏ <62,2%. |

## Ghi nhận AI critique

| Phản biện | Chấp nhận hay bác bỏ | Thay đổi đã thực hiện | Lý do |
|---|---|---|---|
| D30 retention chưa chứng minh retention curve đã phẳng. | CHẤP NHẬN | Đổi North Star thành D30→D60 paid-cohort retention. | Cần quan sát thêm một tháng sau D30 trước khi suy LTV. |
| Containment cao có thể vẫn không tạo giá trị tài chính cho user. | CHẤP NHẬN | Thêm L-04 Confirmed plan value rate và bỏ đèn free-tier khi chính sách free tier chưa chốt. | Day 25 yêu cầu user feedback, confirmed outcome và savings signal. |
| Kill criterion cũ yêu cầu hai kỳ GM 30 ngày quá sớm ở Day 90. | CHẤP NHẬN | Dời mốc kill sang 2026-12-26. | Paid cohort chỉ có đủ hai kỳ chốt sau Day 90. |
