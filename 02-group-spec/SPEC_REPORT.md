# Thin SPEC Cuối Day 05 - AI Delivery Stack & Recovery Copilot

Thin SPEC không phải PRD đầy đủ. Đây là bản cam kết đủ rõ để sáng Day 06 nhóm build prototype ngay.

## 1. Track, product/app và user

**Track:** AI Product / Food Delivery Operations / Customer Support Recovery  
**Sản phẩm/app thật:** ShopeeFood  
**Feature prototype:** AI Delivery Stack & Recovery Copilot  
**User cụ thể:** Khách hàng đặt đồ ăn ShopeeFood trong giờ cao điểm/tối muộn, đang chờ đơn có dấu hiệu trễ hoặc bị kẹt.  
**User phụ:** Support/ops cần nhìn các đơn rủi ro và chọn action recovery; quán/shop và tài xế là bên bị ảnh hưởng bởi quyết định điều phối.  
**Nhóm có phải user thật không? Nếu không, khác ở đâu?** Nhóm có thể là user thật nếu từng đặt ShopeeFood, nhưng chưa có dữ liệu vận hành nội bộ. Evidence hiện tại đến từ 6 ảnh review public/screenshot, cần verify thêm bằng test flow app thật hoặc phỏng vấn nhanh trước M1 Day 06.

## 2. Evidence summary

| Evidence | Nguồn | User/pain nói lên điều gì? | SPEC phải đổi gì? |
|---|---|---|---|
| "Shopee để tài xế ghép quá nhiều đơn, giao 2 tiếng chưa xong..." | `IMG_4017.png` | Pain không chỉ là ETA sai. Vấn đề là hệ thống cho phép ghép đơn quá tải, làm chất lượng đồ ăn và thời gian chờ vượt ngưỡng. | Thêm `driver_active_orders`, `route_detour_minutes`, `food_ready_at`; AI phải có rule `STACK_TOO_DEEP` và action "không ghép thêm/tách batch/tìm tài xế khác". |
| Khách chờ 2 tiếng khi không có tài xế nhận, không được hủy, hỗ trợ chỉ có AI. | `IMG_4019.png`, `IMG_4020.png`, `IMG_4022.png` | Người dùng cần quyền recovery khi hệ thống fail, không chỉ lời xin lỗi hoặc hướng dẫn chung. | Thêm no-driver timeout: Yellow sau 20 phút, Red sau 30 phút; Red phải unlock cancel/refund có điều kiện và escalate người thật. |
| Hotline không liên lạc được, muốn gặp người thật không được, nhân viên kết thúc chat khi chưa giải quyết. | `IMG_4021.png`, `IMG_4022.png` | AI support là ngõ cụt trong case rủi ro cao. Case liên quan tiền, hủy đơn, quán đóng cửa phải có human escalation. | Chọn conditional automation; thêm `support_state`, `HUMAN_SUPPORT_REQUIRED`, rule không cho đóng case khi chưa có resolution. |
| Quán không hủy được, khách cũng không hủy được; 23h tài xế nhận đơn thì quán đã đóng. | `IMG_4022.png` | Các bên không có shared state. Platform cần phát hiện merchant closing risk trước khi điều phối muộn vô nghĩa. | Thêm `merchant_status`, `merchant_closing_time`; AI phải gắn `MERCHANT_CLOSING` và mở platform-mediated cancel path. |
| Người dùng đã bị trừ tiền nhưng đơn báo chưa thanh toán. | `IMG_4021.png` | Payment mismatch là trust risk cao, không thể xử lý bằng chatbot trả lời chung. | Thêm `payment_state = mismatch`; khóa đóng chat, collect evidence, escalate human review. |
| User muốn chat với shop/chặn shop bán kém khi đơn làm sai note hoặc không có số điện thoại liên hệ. | `IMG_4018.png` | Recovery không chỉ là tài xế; cần có merchant correction/log sau trải nghiệm sai. | Đưa merchant communication vào correction path/backlog; prototype tối thiểu có reason `MERCHANT_CONTACT_GAP` và log merchant issue. |

## 3. Pain statement

```text
Khách đặt đồ ăn ShopeeFood đang gặp khó ở bước chờ tài xế/giao hàng,
vì hệ thống ghép đơn/điều phối không nhận diện sớm đơn có nguy cơ trễ nặng
và workflow hủy đơn/hỗ trợ không mở recovery path kịp thời,
dẫn tới việc khách phải chờ 1-2 tiếng, đồ ăn nguội hoặc mất giá trị,
không liên hệ được người thật, quán/tài xế/khách đều bị kẹt,
và niềm tin vào ShopeeFood giảm.

Bằng chứng chính là các review `IMG_4017.png`, `IMG_4019.png`, `IMG_4020.png`, `IMG_4022.png`
về ghép đơn quá nhiều, không có tài xế nhưng không cho hủy, và hỗ trợ AI không giải quyết được.
```

## 4. Build slice

```text
Cho khách đặt đồ ăn ShopeeFood đang chờ đơn có nguy cơ trễ,
prototype sẽ dùng AI để augment quyết định điều phối và hỗ trợ recovery:
tính Delivery Risk Score,
gắn reason code,
giải thích source của rủi ro,
và đề xuất action như không ghép thêm, tách batch, đổi tài xế,
mở hủy/hoàn tiền có điều kiện hoặc chuyển người thật.

Output là:
- Order Risk Monitor cho support/ops,
- AI Decision Panel giải thích risk và action,
- Customer Recovery Preview cho khách,
- Audit/Correction Log cho case đã xử lý.

Prototype xử lý failure mode "không có tài xế sau 30 phút hoặc tài xế bị ghép quá tải"
bằng cách show source, chuyển risk Red, unlock cancel/refund có điều kiện,
và bắt buộc human escalation nếu liên quan thanh toán/quán đóng cửa/tranh chấp.
```

### Phạm vi không build trong Day 06

- Không tích hợp API ShopeeFood thật.
- Không build thuật toán route/dispatch thật.
- Không hoàn tiền thật.
- Không xử lý fraud policy đầy đủ.
- Không build chatbot CSKH hoàn chỉnh.
- Không build merchant portal đầy đủ.

## 5. Auto/Aug decision

Chọn một:

- [ ] **Augmentation:** AI gợi ý/draft/phân loại, user quyết cuối.
- [x] **Conditional automation:** AI tự làm trong case hẹp; case mơ hồ/rủi ro chuyển người.
- [ ] **Automation:** AI tự quyết và tự hành động.

**Lý do chọn:**  
AI có thể tự động tính risk score, gán reason code, hiện cảnh báo, đề xuất action và mở request cancel khi quá ngưỡng. Tuy nhiên, hủy đơn, hoàn tiền, tranh chấp thanh toán, quán đóng cửa và khiếu nại là hành động nhạy cảm, cần người thật làm rescuer/decider.

**Human role:** rescuer + decider + reviewer.  
**AI role:** triage + risk scoring + recommendation + evidence collector + guardrail trigger.

### AI được tự động làm

- Tính `risk_score` và `risk_level`.
- Gán `reason_codes`.
- Hiển thị source của rủi ro.
- Đề xuất `recommended_action`.
- Mở request cancel khi đơn vượt ngưỡng an toàn.
- Ghi audit log/correction log.

### AI không được tự động làm

- Hoàn tiền mọi case.
- Hủy đơn khi chưa đủ điều kiện.
- Đóng ticket liên quan tiền.
- Override merchant/tài xế trong tranh chấp.
- Trả lời chắc chắn khi thiếu dữ liệu.

## 6. Four paths

| Path | Prototype phải thể hiện gì? |
|---|---|
| Happy | Đơn có tài xế trong 5 phút, tài xế chỉ có 1 đơn active, ETA ổn định. AI hiển thị `risk_level = Green`, không cảnh báo quá mức, không mở cancel option quá sớm. |
| Low-confidence | Chưa có tài xế sau 20 phút hoặc tài xế có 2-3 đơn active. AI hiển thị `risk_level = Yellow`, show source như thời gian chờ/số đơn active/ETA, và đưa option: tiếp tục đợi, ưu tiên tìm tài xế, hoặc request cancel nếu tiếp tục vượt ngưỡng. |
| Failure | Chưa có tài xế sau 30 phút, quán sắp đóng cửa, đồ ăn đã ready quá 30 phút, hoặc tài xế có >= 4 đơn active/detour > 25 phút. AI hiển thị `risk_level = Red`, reason code, unlock cancel/refund có điều kiện, đề xuất reassign/tách batch và chuyển người thật. |
| Correction | Payment mismatch, quán đã đóng nhưng hệ thống vẫn điều phối, khách báo món sai note hoặc support muốn đóng chat khi chưa resolve. AI khóa đóng case, ghi correction log, cập nhật reason code và chuyển human review. |

## 7. Failure mode nguy hiểm nhất

```text
Nếu khách đặt đồ ăn vào giờ cao điểm hoặc tối muộn,
AI/hệ thống có thể tiếp tục cho đợi vì không có tài xế hoặc tài xế bị ghép quá nhiều đơn,
trong khi app không cho hủy và support chỉ trả lời tự động,
hậu quả là khách chờ 1-2 tiếng, đồ ăn mất giá trị,
quán/tài xế/khách đều bị kẹt, và ShopeeFood mất trust.

Prototype sẽ xử lý bằng:
- show source của delay,
- Delivery Risk Score,
- reason code rõ ràng,
- ngưỡng unlock cancel/refund,
- human escalation bắt buộc,
- audit log cho mọi action.

Owner kiểm thử path này là: Hoàng Hải.
```

## 8. Data, rules và output cần build

### Input data mock

| Field | Ví dụ | Lý do cần |
|---|---|---|
| `order_created_at` | 21:00 | Tính tổng thời gian khách đã chờ. |
| `driver_assigned_at` | null / 23:00 | Phát hiện no-driver timeout. |
| `food_ready_at` | 21:25 | Tính đồ ăn đã chờ bao lâu sau khi quán làm xong. |
| `current_eta` | 45 phút | So sánh với ETA kỳ vọng. |
| `merchant_status` | open / closing_soon / closed | Tránh điều phối muộn khi quán đã/sắp đóng. |
| `driver_active_orders` | 1 / 3 / 5 | Phát hiện ghép đơn quá tải. |
| `route_detour_minutes` | 8 / 25 / 60 | Đo tác động của việc ghép đơn. |
| `payment_state` | paid / unpaid / mismatch | Phát hiện case đã trừ tiền nhưng đơn báo chưa thanh toán. |
| `support_state` | AI_only / human_pending / human_active | Biết khi nào phải escalate. |
| `customer_cancel_attempts` | 0 / 1 / 3 | Biết khách đã có nhu cầu recovery. |

### Rule tạm thời cho prototype

| Rule | Điều kiện | AI output/action |
|---|---|---|
| No-driver warning | Chưa có tài xế sau 20 phút | `Yellow`, show source, ưu tiên dispatch, cho request cancel. |
| Severe no-driver timeout | Chưa có tài xế sau 30 phút hoặc quá promised ETA | `Red`, unlock cancel/refund có điều kiện, escalate support. |
| Stack overload | Tài xế đang có >= 3 đơn active hoặc detour > 25 phút | Không ghép thêm; đề xuất tài xế khác/tách batch. |
| Food freshness risk | `food_ready_at` quá 30 phút mà chưa pickup/delivery | `Red`, thông báo khách, mở recovery. |
| Merchant closing risk | Quán sắp đóng cửa trong 30 phút và chưa có tài xế | Ưu tiên dispatch hoặc mở platform-mediated cancel. |
| Payment mismatch | Payment đã capture nhưng order báo unpaid | Collect evidence, khóa đóng chat, escalate human. |
| AI support confidence low | Khách hỏi >2 lần hoặc case liên quan tiền/hủy đơn | Chuyển người thật, không lặp câu trả lời chung. |

### AI output

| Output | Giá trị mẫu | Ý nghĩa |
|---|---|---|
| `risk_level` | Green / Yellow / Red | Mức rủi ro của đơn. |
| `risk_score` | 0-100 | Điểm ưu tiên cho support/ops. |
| `reason_codes` | `STACK_TOO_DEEP`, `NO_DRIVER_TIMEOUT`, `FOOD_READY_TOO_LONG`, `MERCHANT_CLOSING`, `PAYMENT_STATE_MISMATCH`, `HUMAN_SUPPORT_REQUIRED` | Lý do AI ra quyết định. |
| `recommended_action` | Do not stack / Reassign driver / Unlock cancel / Escalate human / Merchant check | Hành động đề xuất. |
| `customer_message` | "Đơn của bạn đang trễ vì chưa có tài xế phù hợp. Bạn có thể hủy và nhận hoàn tiền." | Thông điệp minh bạch cho khách. |
| `audit_log` | "21:30 - Risk Red, cancel unlocked" | Dùng cho correction, dispute và học lại. |

## 9. Prototype screens

### Màn hình 1 - Order Risk Monitor

Dashboard cho support/ops thấy danh sách đơn rủi ro:

- order id,
- merchant,
- customer wait time,
- driver assigned/not assigned,
- active stacked orders,
- food ready time,
- merchant status,
- payment state,
- ETA,
- risk score,
- reason codes,
- recommended action.

### Màn hình 2 - AI Decision Panel

Khi chọn một đơn, panel hiển thị:

```text
Risk: Red / 87
Lý do:
- Chưa có tài xế sau 32 phút.
- Quán sẽ đóng cửa sau 18 phút.
- Khách đã bấm hủy 2 lần.

Đề xuất:
1. Mở hủy đơn/hoàn tiền có điều kiện.
2. Ưu tiên tìm tài xế khác trong 5 phút.
3. Chuyển người thật nếu khách cần tranh chấp.
```

### Màn hình 3 - Customer Recovery Preview

Thông điệp khách thấy:

```text
Đơn của bạn đang bị trễ vì hệ thống chưa tìm được tài xế phù hợp.
Bạn đã chờ 32 phút, vượt ngưỡng hỗ trợ của ShopeeFood.

Bạn có thể:
[Hủy đơn và nhận hoàn tiền]
[Tiếp tục đợi, ưu tiên tìm tài xế]
[Gặp nhân viên hỗ trợ]
```

### Màn hình 4 - Audit / Correction Log

```text
21:00 - Đơn được tạo
21:20 - No-driver warning, risk Yellow
21:30 - Risk Red, cancel unlocked
21:31 - User request cancel
21:32 - Human support assigned
21:35 - Refund pending
```

## 10. Test plan

| Test case | Input / tình huống | Expected behavior |
|---|---|---|
| TC01 - Happy path | Có tài xế sau 5 phút, driver có 1 đơn, ETA ổn định. | Risk Green, không hiện cảnh báo/cancel quá sớm. |
| TC02 - No-driver Yellow | Đơn tạo lúc 21:00, 21:20 chưa có driver. | Risk Yellow, show source, đưa option tiếp tục đợi/request cancel, ưu tiên dispatch. |
| TC03 - No-driver Red | Đơn tạo lúc 21:00, 21:30 chưa có driver. | Risk Red, unlock cancel/refund có điều kiện, escalate support. |
| TC04 - Stack overload | Driver có 4 đơn active, đơn mới làm detour tăng 45 phút. | Reason `STACK_TOO_DEEP`, action không ghép thêm/tìm tài xế khác. |
| TC05 - Food ready quá lâu | `food_ready_at` đã qua 40 phút, chưa pickup. | Reason `FOOD_READY_TOO_LONG`, mở recovery và thông báo khách. |
| TC06 - Quán sắp đóng cửa | 22:45 chưa có driver, quán đóng 23:00. | Reason `MERCHANT_CLOSING`, không điều phối muộn vô nghĩa, mở cancel/merchant confirmation. |
| TC07 - Payment mismatch | Payment paid nhưng order state unpaid. | Reason `PAYMENT_STATE_MISMATCH`, khóa đóng chat, escalate human. |
| TC08 - AI support lặp lại | Khách hỏi hủy/hoàn tiền >2 lần. | Reason `HUMAN_SUPPORT_REQUIRED`, chuyển người thật. |

## 11. Owner plan cho sáng Day 06

| Thành viên | Việc phụ trách | Bằng chứng cần có trong repo |
|---|---|---|
| Nguyễn Danh Thành | Research / evidence | 6 ảnh review + `evidence-pack-template.md` + bảng evidence summary. |
| Vũ Hải Dương | SPEC | File `thin-spec-template.md` đã điền, có pain statement, build slice, Auto/Aug, four paths. |
| Vũ Thành Lộc | Prototype | Mock Order Risk Monitor, AI Decision Panel, Customer Recovery Preview, Audit Log. |
| Hoàng Hải | Test / failure path | Test cases TC01-TC08, đặc biệt no-driver Red và stack overload. |
| Vũ Ngọc Vinh + Vũ Tuấn Phương | Demo script / repo | Demo script 3 phút: happy -> low-confidence -> failure -> correction. |

## 12. Demo script 3 phút

```text
1. Mở Order Risk Monitor.
2. Show một đơn Green để chứng minh AI không can thiệp sai.
3. Chuyển sang đơn Yellow: chưa có tài xế sau 20 phút, AI show source và option.
4. Chuyển sang đơn Red: chưa có tài xế sau 32 phút + quán sắp đóng cửa.
5. AI đề xuất mở hủy/hoàn tiền có điều kiện và chuyển người thật.
6. Show Customer Recovery Preview.
7. Show Audit Log để chứng minh case được ghi lại và không biến mất.
```
