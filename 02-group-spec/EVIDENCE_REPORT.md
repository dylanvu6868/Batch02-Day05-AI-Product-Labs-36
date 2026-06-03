# Gói bằng chứng - Tối ưu giao nhận ShopeeFood

Nộp kèm thin SPEC cuối Day 05.

**Ý tưởng chính:** Giải quyết vấn đề ghép đơn dồn dập từ ShopeeFood - tối ưu giao nhận và mở recovery path khi đơn bị kẹt.

## 1. Nhóm và track

**Tên nhóm:** [Điền tên nhóm]  
**Track:** AI Product / Food Delivery Operations / Customer Support Recovery  
**Sản phẩm/app đã chọn:** ShopeeFood  
**Phạm vi build đang nghĩ:** AI Delivery Stack & Recovery Copilot

### Phạm vi build trong 1 ngày

Prototype không cố xây lại toàn bộ hệ thống điều phối tài xế. Phạm vi nên hẹp hơn, tập trung vào điểm gãy có bằng chứng mạnh nhất:

```text
Khi đơn ShopeeFood có rủi ro bị giao trễ do ghép quá nhiều đơn, không tìm được tài xế,
quán sắp đóng cửa, đồ ăn đã chờ quá lâu, hoặc người dùng bị kẹt trong luồng hỗ trợ/hủy đơn,
AI sẽ tính "Delivery Risk Score", giải thích lý do, và gợi ý hành động:

- không ghép thêm đơn / tách batch / đổi tài xế,
- mở quyền hủy đơn hoặc hoàn tiền khi vượt ngưỡng,
- chuyển người thật khi kênh hỗ trợ AI không xử lý được,
- ghi audit log/correction log để hệ thống học từ case sai.
```

### User chính

**Người dùng chính:** Khách hàng đặt đồ ăn trên ShopeeFood, đặc biệt trong khung giờ cao điểm hoặc tối muộn.  
**Người dùng phụ:** Tài xế, quán/shop, nhân viên hỗ trợ.  
**Người bị ảnh hưởng gián tiếp:** Quán ăn bị ảnh hưởng uy tín khi đơn trễ không do quán; tài xế bị ghép đơn quá tải; bộ phận hỗ trợ bị quá tải vì case đã trễ nhưng workflow không cho xử lý nhanh.

### Pain statement ban đầu

```text
Khách hàng đặt đồ ăn đang gặp khó ở bước chờ tài xế/giao hàng,
vì hệ thống ghép đơn hoặc điều phối không nhận diện sớm đơn có nguy cơ trễ nặng,
và workflow hủy đơn/hỗ trợ không mở đường recovery kịp thời,
dẫn tới việc người dùng phải chờ 1-2 tiếng, đồ ăn nguội/mất giá trị,
không liên hệ được người thật, và mất niềm tin vào ShopeeFood.
```

## 2. Bằng chứng tự dùng

Nhóm hiện có bằng chứng từ các ảnh review `.png` trong `E:\AI20K\Day5`. Các ảnh này không phải bằng chứng tự dùng trực tiếp của nhóm, nhưng đủ mạnh để dựng lại workflow thật và định nghĩa các path cần test. Trước checkpoint M1 Day 06, nhóm nên kiểm chứng lại bằng 1-2 đơn thật, quan sát flow hủy đơn/hỗ trợ hiện tại, hoặc phỏng vấn nhanh người từng gặp đơn giao trễ.

| Quan sát | Ảnh/link | Path liên quan | Điều học được |
|---|---|---|---|
| Đơn có thể bị trễ rất lâu khi tài xế bị ghép quá nhiều đơn. Review nói giao 2 tiếng chưa xong tính từ lúc cửa hàng giao đồ. | `IMG_4017.png` | Failure | Vấn đề cốt lõi không chỉ là ETA sai, mà là quá tải do ghép đơn: hệ thống cho phép tài xế nhận/gộp quá nhiều đơn, làm chuỗi giao hàng vượt ngưỡng chấp nhận được đối với đồ ăn. |
| Người dùng chờ lâu vì không có tài xế nhận đơn nhưng app không cho hủy sau 20-30 phút, thậm chí kéo tới vài tiếng. | `IMG_4019.png`, `IMG_4020.png`, `IMG_4022.png` | Failure / Correction | Chính sách hủy đơn và hành vi trong app không khớp kỳ vọng. Nếu không có tài xế trong thời gian dài, app phải tự động mở recovery: hủy, hoàn tiền, đổi quán, ưu tiên điều phối hoặc chuyển người thật hỗ trợ. |
| Kênh hỗ trợ bị cảm nhận là "chỉ có AI", không có người thật, hotline không liên lạc được, nhân viên có thể kết thúc chat khi chưa giải quyết. | `IMG_4019.png`, `IMG_4021.png`, `IMG_4022.png` | Low-confidence / Failure | Hỗ trợ AI không được là ngõ cụt. Case có tiền, thanh toán, đơn kẹt hoặc vượt SLA phải có human escalation rõ ràng. |
| Người dùng không liên hệ được shop khi đơn sai note/món không đúng ý; không có kênh chat với shop hoặc cách chặn shop bán kém. | `IMG_4018.png` | Correction | Tối ưu giao nhận phải bao gồm cả merchant communication/recovery, không chỉ route tài xế. Nếu món sai mà không có kênh liên hệ, người dùng mất quyền sửa lỗi tại điểm phát sinh. |
| Quán/shop cũng bị kẹt: review nói quán không hủy được, khách cũng không hủy được; đến 23h tài xế mới nhận thì quán đã đóng. | `IMG_4022.png` | Failure | Hệ thống đang thiếu shared state giữa khách - shop - tài xế - platform. Khi merchant sắp đóng cửa hoặc đơn trễ quá ngưỡng, app phải tự phát hiện và đồng bộ hành động cho các bên. |
| Đơn/thanh toán có mismatch: người dùng đã bị trừ tiền nhưng đơn báo chưa thanh toán, bộ phận hỗ trợ đùn đẩy trách nhiệm. | `IMG_4021.png` | Correction / Failure | Lệch trạng thái thanh toán là case high-risk. AI chỉ nên thu thập bằng chứng và escalate, không nên trả lời chung hoặc cho phép đóng chat sớm. |

### Bản đồ workflow bị gãy từ bằng chứng

```text
1. Người dùng đặt món.
2. Quán nhận đơn và làm món.
3. Hệ thống tìm tài xế hoặc ghép đơn cho tài xế.
4. Rủi ro xuất hiện:
   - không có tài xế,
   - tài xế bị ghép quá nhiều đơn,
   - quán đã làm xong nhưng đồ ăn chờ quá lâu,
   - quán sắp đóng cửa,
   - thanh toán và trạng thái đơn không khớp.
5. Người dùng muốn hủy đơn, liên hệ hỗ trợ hoặc liên hệ shop.
6. App không mở recovery path đủ rõ:
   - không cho hủy,
   - chủ yếu là hỗ trợ AI,
   - hotline/người thật khó tiếp cận,
   - nhân viên kết thúc chat khi chưa resolve,
   - shop/khách đều không sửa được.
7. Kết quả:
   - người dùng chờ 1-2 tiếng,
   - đồ ăn nguội hoặc không còn giá trị,
   - merchant bị liên lụy,
   - tài xế bị quá tải,
   - niềm tin với ShopeeFood giảm.
```

## 3. Bằng chứng từ người dùng / review / mạng xã hội

Nguồn hiện tại là ảnh review do nhóm tổng hợp trong thư mục `E:\AI20K\Day5`. Một số review có ngôn từ bức xúc; bảng dưới giữ ý chính và chỉ quote phần cần cho product decision.

| Quote / review / quan sát | Nguồn | Người dùng là ai? | Pain/failure mode |
|---|---|---|---|
| "Shopee để tài xế ghép quá nhiều đơn, giao 2 tiếng chưa xong..." | `IMG_4017.png` | Khách đặt đồ ăn bị giao trễ | **Quá tải do ghép đơn:** Tài xế được ghép quá nhiều đơn, làm thời gian giao vượt xa kỳ vọng của đồ ăn. |
| Người dùng dùng hằng ngày yêu cầu thêm chức năng chặn shop bán đồ không ngon, chat với shop khi đơn làm sai note, vì có đơn không kèm số điện thoại nên không biết liên hệ thế nào. | `IMG_4018.png` | Khách hàng thường xuyên | **Khoảng trống giao tiếp với merchant:** Khách không có kênh sửa lỗi với shop, không có feedback/control sau trải nghiệm kém. |
| "Bắt khách hàng chờ 2 tiếng khi order không có tài xế nhận... vào phần hỗ trợ khách hàng không ai giải quyết chỉ có AI." | `IMG_4019.png` | Khách có đơn không tài xế nhận | **No-driver timeout + hỗ trợ chỉ có AI:** App không tự mở hủy đơn, hỗ trợ AI không giải quyết được case quá ngưỡng. |
| Review phản ánh đặt đợi lâu không có tài xế nhưng không cho hủy; đồ giao tới thì không còn giá trị; gọi tổng đài cũng không giải quyết tốt. | `IMG_4020.png` | Khách gặp đơn trễ nghiêm trọng | **Khóa quyền hủy đơn:** Khi đơn đã fail về SLA, người dùng vẫn bị khóa action nên phải tự xử lý ngoài app. |
| Người dùng đã thanh toán/trừ tiền nhưng đơn báo chưa thanh toán; nhân viên hỗ trợ đùn đẩy và kết thúc chat khi chưa giải quyết. | `IMG_4021.png` | Khách gặp lỗi thanh toán/trạng thái đơn | **Lệch trạng thái thanh toán/đơn hàng + đóng hỗ trợ sớm:** Case tiền bạc cần escalation và evidence log, không được đóng chat sớm. |
| "Đặt đơn từ 9h tối... đến 11h đêm vẫn chưa cho hủy... hotline không liên lạc được... muốn liên lạc NGƯỜI THẬT cũng không được... 11h đêm tài xế nhận đơn thì quán đã đóng..." | `IMG_4022.png` | Khách đặt đêm muộn; merchant cũng bị kẹt | **Đơn kẹt ban đêm:** Không có tài xế + không cho hủy + không có người thật + quán đóng cửa tạo failure nghiêm trọng cho cả khách, quán và platform. |

### Evidence themes

| Chủ đề | Ảnh liên quan | Tín hiệu lặp lại | Hàm ý cho sản phẩm |
|---|---|---|---|
| Ghép đơn quá tải | `IMG_4017.png` | Tài xế bị ghép nhiều đơn, giao 2 tiếng | Cần giới hạn/risk score trước khi ghép thêm đơn. |
| Không có tài xế nhận | `IMG_4019.png`, `IMG_4020.png`, `IMG_4022.png` | Người dùng chờ 20-30 phút, 2 tiếng, đến đêm vẫn không được hủy | Cần tự động mở quyền hủy/hoàn tiền hoặc điều phối lại khi quá ngưỡng. |
| Support không đủ recovery | `IMG_4019.png`, `IMG_4021.png`, `IMG_4022.png` | "Chỉ có AI", không liên lạc được người thật, chat bị kết thúc | AI cần có confidence/SLA gate và bắt buộc human escalation. |
| Merchant/contact gap | `IMG_4018.png`, `IMG_4022.png` | Không chat shop, quán không hủy được, quán đóng cửa | Cần đồng bộ trạng thái shop và kênh communication có giới hạn rõ. |
| Rủi ro niềm tin/thanh toán | `IMG_4021.png` | Đã trừ tiền nhưng đơn báo chưa thanh toán | Cần payment mismatch detector và không cho đóng case khi chưa resolve. |

### Định nghĩa user problem sau khi đọc review

Khách hàng không chỉ muốn "giao nhanh hơn". Họ cần:

- biết đơn đang bị kẹt vì lý do gì,
- có quyền hủy/hoàn tiền khi hệ thống vượt ngưỡng,
- được liên hệ người thật khi AI không giải quyết được,
- không bị bắt chờ trong khi quán/tài xế/platform đều không có hành động,
- thấy platform đang bảo vệ chất lượng đồ ăn, không chỉ tối ưu số đơn/tài xế.

## 4. Bằng chứng từ đối thủ / mô hình tương tự

Không dùng browsing trong gói bằng chứng này. Các dòng dưới là pattern tương tự cần kiểm chứng thêm nếu nhóm dùng trong demo/slide.

| App / mô hình tham khảo | Họ xử lý task này thế nào? | Pattern học được | Có áp dụng trong 1 ngày không? |
|---|---|---|---|
| Ride-hailing dispatch | Thường có logic không giao thêm cuốc nếu route/ETA vượt ngưỡng | **Capacity guardrail:** Không tối ưu số job/tài xế bằng mọi giá; cần giới hạn theo SLA và risk. | Có. Prototype bằng rule + risk score UI. |
| Food delivery batching | Đơn có món dễ nguội/đã làm xong lâu nên cần ưu tiên giao hơn đơn mới | **Freshness-aware batching:** Tính thời gian từ lúc shop ready, không chỉ tính khoảng cách. | Có. Mock data có `food_ready_at` và `time_since_ready`. |
| SLA hỗ trợ khách hàng | Case quá ngưỡng thời gian, liên quan tiền, hoặc AI confidence thấp phải escalate | **Cổng chuyển người thật:** AI là triage, không phải điểm kết thúc. | Có. Prototype có nút "Escalate human" và lý do bắt buộc. |
| Payment dispute flow | Khi payment và order state mismatch, case cần lock closure cho đến khi reconcile | **No premature close:** Chat/case không được đóng nếu chưa có resolution status. | Có. Thêm state `payment_mismatch = true`. |
| Merchant operations | Quán cần báo hết món/đóng cửa/không thể chờ lâu để platform action | **Shared state:** Shop, driver, customer cùng nhìn một trạng thái đơn và đường recovery. | Có. Mock merchant status: open/closing/closed. |

## 5. Evidence -> Insight

```text
Evidence nổi bật nhất:
Nhiều review 1 sao lặp lại cùng một mẫu: khách chờ rất lâu do không có tài xế
hoặc tài xế bị ghép quá nhiều đơn, nhưng app không cho hủy, hỗ trợ không có người thật,
và case không được giải quyết kịp thời.

Insight:
Người dùng không chỉ gặp surface problem là "giao hàng chậm".
Thật ra họ cần một cơ chế recovery đáng tin khi đơn đã có dấu hiệu fail:
- minh bạch lý do đơn trễ,
- quyền hủy/hoàn tiền khi hệ thống quá ngưỡng,
- human support khi AI không đủ năng lực,
- bảo vệ chất lượng món ăn sau khi shop đã làm xong,
- đồng bộ giữa khách, tài xế, shop và platform.

Opportunity:
AI có thể giúp bằng cách augment điều phối và hỗ trợ trong một hành động hẹp:
tính Delivery Risk Score cho mỗi đơn/batch, giải thích reason code,
đề xuất hành động tiếp theo, và mở recovery path khi đơn vượt ngưỡng.
```

### Core insight

**ShopeeFood đang có khoảng trống giữa optimization và recovery.**  
Hệ thống có thể đang tối ưu việc gom/gộp đơn để tăng hiệu suất giao nhận, nhưng evidence cho thấy khi optimization sai, người dùng không có đường recover. AI nên được đặt ở điểm này: không phải "chatbot trả lời xin lỗi", mà là **copilot phát hiện đơn sắp fail và kích hoạt hành động đúng lúc**.

### Product opportunity statement

```text
Nếu ShopeeFood có thể nhận diện sớm đơn có nguy cơ fail vì ghép đơn quá tải,
không có tài xế, quán sắp đóng cửa hoặc thanh toán/trạng thái không khớp,
thì app có thể giảm trải nghiệm chờ 1-2 tiếng bằng cách:
1. không ghép thêm đơn rủi ro,
2. tách batch hoặc đổi tài xế,
3. mở hủy đơn/hoàn tiền có điều kiện,
4. đưa hỗ trợ người thật vào đúng case,
5. log lại lý do để cải thiện điều phối về sau.
```

## 6. Evidence đổi SPEC như thế nào?

- [x] Đổi user chính.
- [x] Đổi pain statement.
- [x] Đổi phạm vi build.
- [x] Đổi Auto/Aug decision.
- [x] Đổi 4 paths.
- [x] Đổi failure mode.
- [x] Đổi owner/kế hoạch test.

### Thay đổi quan trọng 1

```text
Trước evidence, nhóm có thể định làm "AI tối ưu giao nhận/ghép đơn" theo hướng tăng hiệu suất.
Sau evidence, nhóm đổi thành "AI Delivery Stack & Recovery Copilot" theo hướng cân bằng hiệu suất và recovery.
Lý do:
Review cho thấy người dùng không chỉ bức xúc vì trễ, mà vì bị khóa action khi đơn đã trễ nặng.
Nếu chỉ tối ưu route mà không có hủy đơn/escalation, prototype vẫn bỏ qua pain lớn nhất.
```

### Thay đổi quan trọng 2

```text
Trước evidence, failure mode có thể được xem là "ETA không chính xác".
Sau evidence, failure mode nguy hiểm nhất là "đơn bị kẹt quá ngưỡng nhưng không ai có quyền sửa":
khách không hủy được, shop không hủy được, hỗ trợ AI không giải quyết được,
hỗ trợ người thật không vào kịp.
Lý do:
IMG_4019, IMG_4020 và IMG_4022 đều lặp lại mẫu chờ 20-30 phút đến hơn 2 tiếng,
không có tài xế/không được hủy/không có người thật.
```

## 7. Đề xuất SPEC từ evidence

### Tên feature

**AI Delivery Stack & Recovery Copilot**

### One-liner

```text
AI phát hiện đơn ShopeeFood có nguy cơ trễ/hỏng trải nghiệm do ghép đơn,
không có tài xế, shop sắp đóng cửa, đồ ăn đã chờ quá lâu, hoặc hỗ trợ bị kẹt;
sau đó đề xuất action điều phối và mở recovery path cho khách.
```

### Auto/Aug decision

**Chọn:** Conditional automation.

Lý do:

- AI có thể tự động tính risk score, phân loại reason code và gợi ý action.
- AI có thể tự động mở một số action an toàn như thông báo delay, hiện lý do, cho phép request cancel khi quá ngưỡng.
- AI không nên tự động hủy/hoàn tiền mọi case vì có rủi ro gian lận, ảnh hưởng merchant và tài xế.
- Case payment mismatch, tranh chấp, shop đóng cửa hoặc tiền đã trừ cần human review.

**Vai trò của người thật:** rescuer + decider cho case high-risk.  
**AI role:** triage + risk scoring + recommendation + evidence collector + guardrail trigger.

### Data đầu vào cần có trong prototype

| Field | Ví dụ | Lý do cần |
|---|---|---|
| `order_created_at` | 21:00 | Tính tổng thời gian người dùng đã chờ. |
| `driver_assigned_at` | null / 23:00 | Phát hiện no-driver timeout. |
| `food_ready_at` | 21:25 | Tính đồ ăn đã chờ bao lâu sau khi shop làm xong. |
| `current_eta` | 45 phút | So sánh với promised ETA. |
| `merchant_status` | open / closing_soon / closed | Tránh case tài xế nhận khi quán đã đóng. |
| `driver_active_orders` | 1 / 3 / 5 | Phát hiện ghép đơn quá tải. |
| `route_detour_minutes` | 8 / 25 / 60 | Đo tác động của ghép đơn vào đơn hiện tại. |
| `payment_state` | paid / unpaid / mismatch | Phát hiện case đã trừ tiền nhưng order báo chưa thanh toán. |
| `support_state` | AI_only / human_pending / human_active | Biết khi nào phải escalate. |
| `customer_cancel_attempts` | 0 / 1 / 3 | Biết người dùng đã có nhu cầu recovery. |

### Output của AI

| Output | Giá trị mẫu | Dùng để làm gì |
|---|---|---|
| `risk_level` | Green / Yellow / Red | Hiển thị mức rủi ro của đơn/batch. |
| `risk_score` | 0-100 | Sắp xếp ưu tiên trong support/dispatch queue. |
| `reason_codes` | `STACK_TOO_DEEP`, `NO_DRIVER_TIMEOUT`, `FOOD_READY_TOO_LONG`, `MERCHANT_CLOSING`, `PAYMENT_STATE_MISMATCH`, `HUMAN_SUPPORT_REQUIRED` | Giải thích vì sao AI đưa ra đề xuất. |
| `recommended_action` | Do not stack / Reassign driver / Unlock cancel / Escalate human / Merchant check | Biến insight thành action. |
| `customer_message` | "Đơn của bạn đang trễ vì chưa có tài xế phù hợp. Bạn có thể hủy và nhận hoàn tiền." | Minh bạch với người dùng. |
| `audit_log` | "21:35 no-driver timeout, cancel unlocked" | Phục vụ correction và dispute. |

### Rule ngưỡng tạm thời cho prototype

Đây là ngưỡng để demo, cần validate bằng data thật:

| Rule | Điều kiện | Action đề xuất |
|---|---|---|
| No-driver timeout | Chưa có tài xế sau 20 phút | Hiện cảnh báo Yellow, cho request cancel, đưa vào queue dispatch ưu tiên. |
| Severe no-driver timeout | Chưa có tài xế sau 30 phút hoặc quá promised ETA | Red, unlock cancel/refund có điều kiện, escalate support. |
| Stack overload | Tài xế đang có >= 3 đơn active hoặc route detour > 25 phút | Không ghép thêm đơn; đề xuất tài xế khác/tách batch. |
| Food freshness risk | `food_ready_at` quá 30 phút mà chưa pickup/delivery | Red nếu món ăn dễ nguội; thông báo và mở recovery. |
| Merchant closing risk | Merchant sắp đóng cửa trong 30 phút và chưa có tài xế | Ưu tiên dispatch hoặc unlock cancel trước khi quán đóng. |
| Payment mismatch | Payment đã capture nhưng order báo unpaid | Auto collect evidence, khóa việc đóng chat, escalate human. |
| AI support confidence low | Người dùng đã hỏi >2 lần hoặc case liên quan tiền/hủy đơn | Human support required. |

## 8. Four paths cho prototype

| Path | Prototype phải thể hiện gì? | Evidence gốc | Acceptance criteria |
|---|---|---|---|
| Happy | Đơn có 1 tài xế, batch hợp lý, ETA ổn định. AI hiện risk Green và không cần can thiệp. | Dùng làm baseline để so sánh với review xấu. | Người dùng thấy ETA rõ, không bị show cảnh báo sai. |
| Low-confidence | AI thấy risk Yellow: tài xế có nhiều đơn hoặc chưa có tài xế gần ngưỡng. AI hỏi/đưa option: tiếp tục đợi, tìm tài xế khác, hoặc request cancel nếu quá ngưỡng. | `IMG_4017.png`, `IMG_4019.png` | AI không nói chắc chắn khi data thiếu; hiện source như thời gian đợi, số đơn active, ETA. |
| Failure | Risk Red: không có tài xế sau 30 phút, quán sắp đóng cửa, food ready quá lâu, hoặc tài xế bị ghép quá tải. | `IMG_4019.png`, `IMG_4020.png`, `IMG_4022.png` | App mở recovery path: unlock cancel/refund, reassign/tách batch, escalate human. Không chỉ xin lỗi. |
| Correction | Người dùng/shop/support báo state sai: đã thanh toán nhưng order unpaid, quán đã đóng, món sai note, tài xế nhận quá muộn. | `IMG_4018.png`, `IMG_4021.png`, `IMG_4022.png` | AI log correction, cập nhật reason code, không cho đóng case khi chưa resolve, có audit trail. |

## 9. Failure mode nguy hiểm nhất

```text
Nếu người dùng đặt đồ ăn vào giờ cao điểm/tối muộn,
AI/hệ thống có thể tiếp tục cho đợi vì không có tài xế hoặc ghép đơn quá tải,
trong khi app không cho hủy và support chỉ trả lời tự động,
hậu quả là người dùng chờ 1-2 tiếng, đồ ăn mất giá trị,
quán/tài xế/khách đều bị kẹt, và ShopeeFood mất trust.

Prototype sẽ xử lý bằng:
- show source của delay,
- Delivery Risk Score,
- ngưỡng unlock cancel/refund,
- human escalation bắt buộc,
- audit log cho mọi action.

Owner kiểm thử path này là: Hoàng Hải.
```

## 10. Test cases để gắn vào SPEC

| Test case | Input / tình huống | Expected behavior |
|---|---|---|
| TC01 - Ghép đơn quá tải | Tài xế đang có 4 đơn active, đơn mới nếu ghép sẽ tăng detour 45 phút. | AI gán `risk_level = Red`, reason `STACK_TOO_DEEP`, action "không ghép thêm / tìm tài xế khác". |
| TC02 - Giao trễ sau khi shop đã làm xong | `food_ready_at` đã qua 40 phút, tài xế vẫn đang giao đơn khác. | AI gán `FOOD_READY_TOO_LONG`, thông báo cho người dùng, đưa option cancel/refund hoặc support. |
| TC03 - Không có tài xế sau 20 phút | Đơn tạo lúc 21:00, đến 21:20 chưa có driver. | AI gán Yellow, đưa option tiếp tục đợi/request cancel, đưa đơn vào dispatch priority. |
| TC04 - Không có tài xế sau 30 phút | Đơn tạo lúc 21:00, đến 21:30 chưa có driver. | AI gán Red, unlock cancel/refund có điều kiện và escalate support nếu người dùng cần. |
| TC05 - Quán sắp đóng cửa | Đơn tạo 21:00, đến 22:45 chưa có driver, merchant closing at 23:00. | AI gán `MERCHANT_CLOSING`, không cho dispatch muộn vô nghĩa, mở cancel/merchant confirmation. |
| TC06 - Payment mismatch | Người dùng đã bị trừ tiền nhưng order state là unpaid. | AI gán `PAYMENT_STATE_MISMATCH`, collect transaction evidence, khóa việc đóng chat, escalate human. |
| TC07 - AI support không giải quyết | Người dùng hỏi về hủy đơn/hoàn tiền quá 2 lần, AI không có action. | Hệ thống bắt buộc chuyển human support, hiện SLA dự kiến, không lặp lại câu trả lời chung. |
| TC08 - Shop communication gap | Người dùng báo shop làm sai note và không có số điện thoại/chat. | AI hiện option report merchant, request correction/refund, block shop trong preference, log merchant issue. |
| TC09 - Quán và khách đều không hủy được | Merchant status "cannot cancel", customer cancel locked, no driver. | AI mở platform-mediated cancel path, không đẩy trách nhiệm cho một bên. |
| TC10 - Happy path không bị can thiệp sai | Đơn có tài xế trong 5 phút, driver có 1 đơn, ETA đúng ngưỡng. | AI risk Green, không hiện cảnh báo/cancel option quá sớm. |

## 11. Metric nên đưa vào demo/slide

| Chỉ số | Định nghĩa | Vì sao liên quan evidence |
|---|---|---|
| No-driver wait time | Thời gian từ lúc order created đến khi có driver | Review lặp lại việc không có tài xế nhưng không được hủy. |
| Food-ready wait time | Thời gian từ lúc shop làm xong đến khi pickup/delivery | `IMG_4017.png` nói trễ từ lúc cửa hàng giao đồ. |
| Active stacked orders per driver | Số đơn active tài xế đang cầm | Do batching overload là pain chính. |
| Route detour minutes | Số phút tăng thêm do ghép đơn | Khách không quan tâm "có tài xế" nếu tài xế đi vòng qua quá nhiều đơn. |
| Cancel unlock latency | Thời gian từ khi quá ngưỡng đến khi app cho hủy | Review phản ánh không được hủy sau 20-30 phút. |
| Human escalation latency | Thời gian từ khi AI fail đến khi có người thật | Review phản ánh chỉ có AI/hotline không liên lạc/không người thật. |
| Support premature close rate | Tỷ lệ chat/case bị đóng khi chưa có resolution | `IMG_4021.png` nói nhân viên kết thúc chat khi chưa giải quyết. |
| Payment mismatch resolution time | Thời gian resolve case đã trừ tiền nhưng order unpaid | Pain liên quan tiền có trust risk cao. |

## 12. Prototype UI nên có

### Màn hình 1 - Order Risk Monitor

Hiển thị danh sách đơn đang có rủi ro:

- order id,
- merchant,
- customer wait time,
- driver assigned/not assigned,
- active stacked orders,
- food ready time,
- ETA,
- risk score,
- reason codes,
- recommended action.

### Màn hình 2 - AI Decision Panel

Khi bấm vào một đơn, AI giải thích:

```text
Risk: Red / 87
Lý do:
- Chưa có tài xế sau 32 phút.
- Quán sẽ đóng cửa sau 18 phút.
- Khách đã bấm hủy 2 lần.

Đề xuất:
1. Mở cancel/refund cho khách.
2. Gửi thông báo xin lỗi + lý do rõ ràng.
3. Chuyển human support nếu khách cần tranh chấp.
```

### Màn hình 3 - Customer Recovery Preview

Thông điệp người dùng thấy trong app:

```text
Đơn của bạn đang bị trễ vì hệ thống chưa tìm được tài xế phù hợp.
Bạn đã chờ 32 phút, vượt ngưỡng hỗ trợ của ShopeeFood.

Bạn có thể:
[Hủy đơn và nhận hoàn tiền]
[Tiếp tục đợi, ưu tiên tìm tài xế]
[Gặp nhân viên hỗ trợ]
```

### Màn hình 4 - Audit / Correction Log

Log mẫu:

```text
21:00 - Đơn được tạo
21:20 - No-driver warning, risk Yellow
21:30 - Risk Red, cancel unlocked
21:31 - User request cancel
21:32 - Human support assigned
21:35 - Refund pending
```

## 13. Owner plan cho sáng Day 06

| Thành viên | Việc phụ trách | Bằng chứng cần có trong repo |
|---|---|---|
| Nguyễn Danh Thành | Research / evidence | Thư mục ảnh review + bảng evidence map trong file này. |
| Vũ Hải Dương | SPEC | Thin SPEC có pain statement, Auto/Aug decision, four paths. |
| Vũ Thành Lộc | Prototype UI | Mock Order Risk Monitor + AI Decision Panel. |
| Hoàng Hải | Test / failure path | Test cases TC01-TC10, đặc biệt no-driver timeout và support escalation. |
| Vũ Ngọc Vinh + Vũ Tuấn Phương | Demo script / repo | Script demo 3 phút: happy -> low-confidence -> failure -> correction. |

## 14. Các giả định cần verify

```text
Đây là evidence từ review screenshot, chưa phải dataset vận hành nội bộ của ShopeeFood.
Nhóm sẽ kiểm bằng:
1. test flow đặt đơn thật trong app nếu có điều kiện,
2. phỏng vấn nhanh 2-3 người từng gặp đơn trễ/hủy đơn,
3. search thêm review public về "ShopeeFood không hủy đơn", "không có tài xế", "ghép đơn",
4. so sánh với flow support/hủy đơn hiện tại trong app,
5. validate ngưỡng 20/30 phút với expectation của user.
```

## 15. Kết luận gói bằng chứng

Bằng chứng từ 6 ảnh review đang chỉ về cùng một opportunity: **tối ưu giao nhận không thể chỉ là ghép được nhiều đơn hơn**. Với food delivery, tối ưu đúng phải biết khi nào không nên ghép thêm, khi nào phải ưu tiên đồ ăn đã sẵn sàng, khi nào phải mở quyền hủy, và khi nào hỗ trợ AI phải chuyển người thật.

Phạm vi build nên tập trung vào **Delivery Risk Score + Recovery Action**, vì đây là điểm biến evidence thành SPEC rõ nhất:

- người dùng thấy lý do đơn trễ,
- hệ thống có ngưỡng để không ghép đơn quá tải,
- bộ phận hỗ trợ có reason code để xử lý nhanh,
- merchant/driver/customer có chung một trạng thái,
- case sai có correction log để học lại.
