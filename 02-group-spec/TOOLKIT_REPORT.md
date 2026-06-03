# Toolkit - Từ Evidence Đến Build Slice

Dùng sau khi nhóm đã có evidence. Mục tiêu của file này là chốt một build slice đủ nhỏ để sáng Day 06 nhóm có thể build prototype ngay.

**Sản phẩm/app:** ShopeeFood  
**Ý tưởng:** Giải quyết vấn đề ghép đơn dồn dập - tối ưu giao nhận và mở recovery path khi đơn bị kẹt.  
**Feature đề xuất:** AI Delivery Stack & Recovery Copilot  
**Nguồn evidence chính:** `IMG_4017.png` đến `IMG_4022.png` và file `evidence-pack-template.md`.

## 1. Gom evidence thành cụm

Gom theo workflow/pain, không gom theo tên feature.

| Cụm evidence | Evidence liên quan | Pattern lặp lại | Pain thật sự | Path liên quan |
|---|---|---|---|---|
| Ghép đơn quá tải | `IMG_4017.png` | Tài xế bị ghép quá nhiều đơn, giao 2 tiếng chưa xong từ lúc cửa hàng giao đồ. | Hệ thống tối ưu hiệu suất giao nhận nhưng thiếu guardrail về chất lượng món ăn và thời gian chờ của khách. | Failure |
| Không có tài xế nhưng không cho hủy | `IMG_4019.png`, `IMG_4020.png`, `IMG_4022.png` | Khách chờ 20-30 phút đến hơn 2 tiếng, không có tài xế nhưng app vẫn khóa hủy đơn. | Người dùng không chỉ muốn biết ETA; họ cần quyền recovery khi hệ thống đã quá ngưỡng. | Failure / Correction |
| Hỗ trợ AI là ngõ cụt | `IMG_4019.png`, `IMG_4021.png`, `IMG_4022.png` | Review nói chỉ có AI, hotline không liên lạc được, nhân viên kết thúc chat khi chưa giải quyết. | Hỗ trợ AI không được là điểm kết thúc cho case rủi ro cao như tiền, đơn kẹt, hủy đơn, quán đóng cửa. | Low-confidence / Failure |
| Thiếu liên lạc với shop/merchant | `IMG_4018.png`, `IMG_4022.png` | Người dùng muốn chat với shop, chặn shop bán kém; quán và khách đều không hủy được; tài xế nhận khi quán đã đóng. | Các bên không có shared state; khi đơn fail, khách, quán, tài xế và platform không nhìn cùng một sự thật. | Correction / Failure |
| Lệch trạng thái thanh toán | `IMG_4021.png` | Người dùng đã bị trừ tiền nhưng đơn báo chưa thanh toán, hỗ trợ đùn đẩy và đóng chat. | Đây là trust risk cao; case liên quan tiền phải có evidence log và human escalation. | Correction |

### Cụm pain ưu tiên

**Cụm nên build:** Không có tài xế/ghép đơn quá tải dẫn đến giao trễ, nhưng app không mở recovery path đúng lúc.

Lý do chọn cụm này:

- Có evidence mạnh nhất và lặp lại ở nhiều ảnh review.
- Có thể demo được trong 3-5 phút bằng mock data.
- AI có vai trò rõ: phát hiện rủi ro, giải thích lý do, gợi ý action, mở fallback.
- Có đủ 4 path: happy, low-confidence, failure, correction.
- Có thể mở rộng sang merchant/payment/support sau demo.

## 2. Viết insight

```text
Khách đặt đồ ăn trên ShopeeFood không chỉ cần "giao nhanh hơn".
Họ thật ra cần một cơ chế recovery đáng tin khi đơn đã có dấu hiệu fail,
vì nhiều review cho thấy khách bị bắt chờ 1-2 tiếng do không có tài xế
hoặc tài xế bị ghép quá nhiều đơn, nhưng app không cho hủy,
hỗ trợ chủ yếu là AI và không có người thật vào kịp.
```

### Insight đã chốt

**ShopeeFood đang có khoảng trống giữa optimization và recovery.**

Hệ thống có thể tối ưu ghép đơn để tăng hiệu suất, nhưng khi tối ưu sai, user không có đường recover. Vì vậy AI không nên chỉ là chatbot xin lỗi; AI nên là copilot phát hiện đơn sắp fail, giải thích lý do, và kích hoạt hành động đúng lúc.

## 3. Viết opportunity

```text
Cơ hội là dùng AI để augment điều phối và hỗ trợ trong một hành động hẹp:
tính Delivery Risk Score cho mỗi đơn/batch,
giải thích reason code,
đề xuất action như không ghép thêm, tách batch, đổi tài xế, mở hủy/hoàn tiền,
và chuyển người thật khi case vượt ngưỡng,
trong khi vẫn kiểm soát rủi ro gian lận, thanh toán sai và quyết định hủy/hoàn tiền nhạy cảm.
```

### Opportunity statement

Nếu ShopeeFood nhận diện sớm đơn có nguy cơ fail vì ghép đơn quá tải, không có tài xế, đồ ăn đã chờ quá lâu, quán sắp đóng cửa hoặc hỗ trợ AI bị kẹt, app có thể giảm trải nghiệm chờ 1-2 tiếng bằng cách:

- không ghép thêm đơn rủi ro,
- tách batch hoặc đổi tài xế,
- mở quyền hủy/hoàn tiền có điều kiện,
- đưa người thật vào đúng case,
- ghi audit log để support và vận hành học lại từ case sai.

## 4. Chọn build slice

Build slice tốt phải qua 5 câu hỏi.

| Câu hỏi | Đạt chưa? | Quyết định cho nhóm |
|---|---|---|
| User cụ thể chưa? | Đạt | Khách đặt đồ ăn ShopeeFood trong giờ cao điểm/tối muộn, đang chờ đơn có dấu hiệu trễ. Secondary user là support/ops dùng dashboard. |
| Task đủ hẹp chưa? | Đạt | Chỉ demo một luồng: đơn có nguy cơ fail -> AI tính risk -> đề xuất action -> mở recovery path. |
| AI decision rõ chưa? | Đạt | AI tính `risk_score`, gán `risk_level`, gắn `reason_codes`, đề xuất `recommended_action`. |
| Failure path rõ chưa? | Đạt | Không có tài xế sau 30 phút hoặc tài xế đang có quá nhiều đơn, app phải mở hủy/hoàn tiền hoặc chuyển người thật. |
| Có evidence không? | Đạt | 6 ảnh review chỉ ra ghép đơn quá tải, không cho hủy, hỗ trợ AI là ngõ cụt, quán/khách đều bị kẹt. |

### Build slice đã chốt

```text
AI Delivery Stack & Recovery Copilot

Cho đơn ShopeeFood đang có nguy cơ giao trễ,
prototype sẽ dùng AI để tính Delivery Risk Score,
giải thích vì sao đơn rủi ro,
và đề xuất hành động điều phối/recovery:
- không ghép thêm đơn,
- tìm tài xế khác hoặc tách batch,
- mở quyền hủy/hoàn tiền khi quá ngưỡng,
- chuyển người thật khi liên quan thanh toán, quán đóng cửa hoặc AI không chắc.
```

### Không build trong Day 06

Nhóm không build các phần sau trong prototype ngày đầu:

- Không build thuật toán dispatch thật kết nối tài xế thật.
- Không tích hợp API ShopeeFood thật.
- Không tự động hoàn tiền thật.
- Không xử lý fraud/risk policy đầy đủ.
- Không build chatbot support hoàn chỉnh.
- Không build toàn bộ merchant portal.

## 5. Quyết định: giữ, giảm scope, hay đổi hướng?

| Tình huống | Quyết định | Lý do |
|---|---|---|
| Evidence mạnh nhưng ý tưởng ban đầu rộng | Giữ domain, giảm scope xuống một flow | "Tối ưu giao nhận" quá rộng; Day 06 chỉ build risk scoring + recovery action. |
| AI có thể tự động nhưng rủi ro cao | Chọn conditional automation | Hủy/hoàn tiền/thanh toán là hành động nhạy cảm, cần người thật cho case high-risk. |
| Có nhiều pain phụ như shop, thanh toán, support | Đưa vào reason code/backlog | Prototype chính tập trung no-driver + stack overload, nhưng vẫn show được payment/merchant như correction path. |
| Không demo được dispatch thật | Dùng mock data + dashboard | Demo tập trung vào quyết định sản phẩm, không cần vận hành thật. |
| User bị ảnh hưởng nhiều bên | Thiết kế shared state | Dashboard cần hiện cùng lúc trạng thái khách, tài xế, shop, payment, support. |

### Auto/Aug decision

**Chọn:** Conditional automation.

AI được tự động:

- tính risk score,
- phân loại reason code,
- hiện cảnh báo,
- đề xuất hành động,
- mở request cancel khi quá ngưỡng,
- tạo audit log.

AI không được tự động:

- hoàn tiền mọi case,
- hủy đơn khi chưa đủ điều kiện,
- đóng ticket liên quan tiền,
- override merchant/tài xế trong tranh chấp.

Người thật giữ vai trò:

- rescuer khi case quá SLA,
- decider cho hoàn tiền/hủy đơn nhạy cảm,
- reviewer cho payment mismatch,
- trainer để cải thiện rule sau demo.

## 6. Câu chốt cuối

```text
Dựa trên 6 ảnh review về ShopeeFood cho thấy khách bị giao trễ do ghép đơn quá tải,
không có tài xế nhưng không được hủy, và hỗ trợ AI không giải quyết được,
nhóm sẽ build prototype "AI Delivery Stack & Recovery Copilot"
cho khách đặt đồ ăn đang chờ đơn có nguy cơ trễ,
để giải quyết pain đơn bị kẹt quá ngưỡng mà không có đường recovery,
bằng cách AI tính Delivery Risk Score, show lý do, đề xuất action điều phối/hủy/hoàn tiền,
và sẽ test failure path "không có tài xế sau 30 phút hoặc tài xế bị ghép quá nhiều đơn".
```

## 7. Backlog

Những thứ không build trong Day 06:

- Tích hợp dữ liệu ShopeeFood thật.
- Mô hình tối ưu route/dispatch theo bản đồ thật.
- Hệ thống refund/fraud policy đầy đủ.
- Merchant portal cho quán xác nhận đóng cửa, hết món, không thể chờ.
- Driver app flow để tài xế từ chối batch quá tải.
- Chatbot CSKH đa lượt.
- Hệ thống chặn shop/báo shop bán kém theo preference người dùng.
- A/B testing ngưỡng 20 phút/30 phút bằng dữ liệu vận hành thật.

## 8. Giải pháp prototype đề xuất

### Màn hình 1 - Order Risk Monitor

Dashboard cho support/ops thấy các đơn đang có rủi ro:

- mã đơn,
- quán,
- thời gian khách đã chờ,
- trạng thái tài xế,
- số đơn active của tài xế,
- thời gian đồ ăn đã sẵn sàng,
- trạng thái quán,
- trạng thái thanh toán,
- risk score,
- reason code,
- recommended action.

### Màn hình 2 - AI Decision Panel

Khi chọn một đơn, AI giải thích:

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

Thông điệp phía khách:

```text
Đơn của bạn đang bị trễ vì hệ thống chưa tìm được tài xế phù hợp.
Bạn đã chờ 32 phút, vượt ngưỡng hỗ trợ của ShopeeFood.

Bạn có thể:
[Hủy đơn và nhận hoàn tiền]
[Tiếp tục đợi, ưu tiên tìm tài xế]
[Gặp nhân viên hỗ trợ]
```

### Màn hình 4 - Audit / Correction Log

Log cho support và vận hành:

```text
21:00 - Đơn được tạo
21:20 - No-driver warning, risk Yellow
21:30 - Risk Red, cancel unlocked
21:31 - User request cancel
21:32 - Human support assigned
21:35 - Refund pending
```

## 9. Test path bắt buộc

| Path | Tình huống demo | Expected behavior |
|---|---|---|
| Happy | Có tài xế sau 5 phút, tài xế chỉ có 1 đơn, ETA ổn định. | Risk Green, không cảnh báo quá mức. |
| Low-confidence | Chưa có tài xế sau 20 phút hoặc tài xế có 2-3 đơn active. | Risk Yellow, AI show source và đưa option tiếp tục đợi/tìm tài xế/request cancel. |
| Failure | Chưa có tài xế sau 30 phút hoặc tài xế có 4 đơn active, detour 45 phút. | Risk Red, unlock cancel/refund, đề xuất reassign/tách batch, escalate người thật. |
| Correction | Payment mismatch hoặc quán đã đóng nhưng hệ thống vẫn điều phối. | AI khóa đóng case, log correction, chuyển người thật. |
