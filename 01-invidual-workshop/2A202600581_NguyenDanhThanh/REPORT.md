# Report App Teardown: Vietnam Airlines NEO

WebHTML Analysis: `index.html`

## 0. Thông tin nhanh

**Sản phẩm:** Vietnam Airlines  
**AI feature:** NEO - chatbot hỗ trợ vé máy bay, hành lý, tình trạng chuyến bay và các yêu cầu dịch vụ  
**Case dùng thử:** Đánh giá NEO qua 3 truy vấn thực tế về đổi vé, delay chuyến bay và hành lý đặc biệt  
**Inputs quan sát:**

1. "Đổi vé VN257 ngày mai từ Hà Nội đi TP.HCM mất bao nhiêu tiền?"
2. "Tôi đang ở sân bay Nội Bài, chuyến VN154 bị delay bao lâu?"
3. "Tôi mang xe đạp thể thao carbon lên máy bay được không?"

**Evidence:** Nội dung phân tích trong `index.html` và các ảnh chụp màn hình trong thư mục `2A202600581_NguyenDanhThanh`.

Mục tiêu của teardown này là tìm khoảng cách giữa kỳ vọng của người dùng và phản hồi thực tế của chatbot NEO. Kết quả cho thấy NEO nhận diện intent khá tốt và có hành vi an toàn, nhưng nhiều luồng vẫn dừng ở mức trả lời/ hỏi thêm thông tin thay vì hoàn thành tác vụ cho user.

## 1. Product được chọn

| Hạng mục | Nội dung |
|---|---|
| Product | Vietnam Airlines |
| AI feature | NEO - chatbot hỗ trợ khách hàng |
| User chính trong case | Hành khách cần hỗ trợ nhanh về vé bay, chuyến bay, hành lý |
| Task kỳ vọng | Hỏi bằng ngôn ngữ tự nhiên để được xử lý tác vụ: tính phí đổi vé, tra delay realtime, xác nhận quy định hành lý |

## 2. Promise vs Reality

### Promise

NEO tạo kỳ vọng rằng người dùng có thể hỏi trực tiếp các vấn đề liên quan đến hành trình bay và nhận được hỗ trợ rõ ràng, nhanh, đúng ngữ cảnh. Với các câu hỏi như "mất bao nhiêu tiền", "delay bao lâu", "có mang được không", user không chỉ muốn đọc thông tin FAQ mà muốn có câu trả lời có thể hành động ngay.

User có thể kỳ vọng NEO làm được các việc sau:

- Tra cứu booking hoặc chuyến bay dựa trên mã chuyến bay, ngày bay, hành trình.
- Ước tính hoặc tính chính xác phí đổi vé khi có đủ thông tin.
- Trả tình trạng chuyến bay realtime: delay bao lâu, giờ bay mới, gate, boarding.
- Trả lời quy định hành lý đặc biệt bằng kết luận ngắn gọn và các bước cần làm.
- Nếu thiếu dữ liệu, hỏi lại bằng form/options thay vì để user tiếp tục chat tự do.

### Reality

Trong 3 query quan sát, NEO xử lý khá tốt ở tầng nhận diện ý định và an toàn dữ liệu. Bot không tự bịa phí đổi vé, không tự suy diễn thời gian delay khi thiếu dữ liệu, và có thể trả lời policy hành lý đặc biệt khá đầy đủ.

Tuy nhiên, điểm gãy lớn là NEO chưa chuyển tốt từ "trả lời thông tin" sang "hoàn thành tác vụ":

- Với đổi vé, bot yêu cầu mã đặt chỗ nhưng chưa đưa user vào flow đổi vé có cấu trúc.
- Với delay chuyến bay, bot xác nhận lại thông tin nhưng user đang ở sân bay cần câu trả lời realtime nhanh hơn.
- Với xe đạp carbon, bot trả lời đủ policy nhưng hơi dài so với nhu cầu "có được mang hay không".
- Các flow thiếu suggested actions, handoff, correction path và trạng thái tiếp theo rõ ràng.

### Impact

User vẫn phải tự tiếp tục nhập thông tin, tự tìm trang dịch vụ, hoặc tự đọc policy dài để rút ra quyết định. Trong ngữ cảnh sân bay hoặc thay đổi vé, việc chậm hoàn thành tác vụ có thể làm user mất thời gian, lo lắng và tăng khả năng phải liên hệ nhân viên hỗ trợ.

## 3. Evidence quan sát được

### Query 1: Đổi vé

**Input:** "Đổi vé VN257 ngày mai từ Hà Nội đi TP.HCM mất bao nhiêu tiền?"

**Phản hồi thực tế:** NEO yêu cầu user cung cấp mã đặt chỗ để kiểm tra chính xác.

**Điểm mạnh:** Bot nhận diện đúng intent đổi vé và không suy đoán chi phí khi thiếu dữ liệu cá nhân hóa.

**Điểm gãy:** User hỏi về chi phí đổi vé, nhưng bot mới dừng ở bước thu thập thông tin. Chưa có form nhập mã đặt chỗ, chưa có nút "đổi ngày / đổi giờ / đổi hành trình", chưa giải thích rõ các yếu tố ảnh hưởng đến phí.

### Query 2: Delay chuyến bay

**Input:** "Tôi đang ở sân bay Nội Bài, chuyến VN154 bị delay bao lâu?"

**Phản hồi thực tế:** NEO xác nhận lại số hiệu chuyến bay và ngày khởi hành trước khi tra cứu.

**Điểm mạnh:** Bot có bước xác thực thông tin trước khi truy vấn, tránh trả lời sai.

**Điểm gãy:** User đang ở sân bay nên cần thông tin thời gian thực. Nếu ngày hiện tại có thể suy luận được, bot nên trả nhanh tình trạng chuyến bay và chỉ hỏi lại khi có nhiều khả năng gây nhầm lẫn.

### Query 3: Xe đạp carbon

**Input:** "Tôi mang xe đạp thể thao carbon lên máy bay được không?"

**Phản hồi thực tế:** NEO trả lời đầy đủ quy định hành lý đặc biệt.

**Điểm mạnh:** Nội dung policy tương đối chính xác và đầy đủ: có thể vận chuyển dưới dạng hành lý ký gửi, cần đóng gói đúng quy chuẩn, nên đăng ký trước, có giới hạn kích thước/trọng lượng và có thể phát sinh phí.

**Điểm gãy:** Câu trả lời dài so với nhu cầu chính của user. Câu trả lời nên bắt đầu bằng kết luận rõ: "Có, nhưng phải ký gửi và đóng gói theo quy định."

## 4. Bốn paths

| Path | Hiện trạng quan sát | Kỳ vọng tốt hơn |
|---|---|---|
| Happy path | NEO nhận diện đúng intent ở cả 3 case. Với hành lý đặc biệt, bot trả lời đủ policy. | Khi đủ dữ liệu, NEO trả kết quả trực tiếp theo cấu trúc: kết luận, chi tiết cần biết, action tiếp theo. |
| Low-confidence path | Với đổi vé, bot yêu cầu mã đặt chỗ. Với delay, bot xác nhận lại thông tin. Tuy nhiên các bước này vẫn là chat tự do. | Khi thiếu dữ liệu, NEO hỏi lại bằng form/options ngắn: mã đặt chỗ, ngày bay, loại đổi vé, xác nhận chuyến bay. |
| Failure path | Nếu chưa có dữ liệu realtime hoặc booking, bot để user tiếp tục nhập thay vì đưa đến path hoàn thành tác vụ. | Nếu không trả được kết quả, bot cần nói rõ lý do, đưa link/deep link đến trang đổi vé, trang tình trạng chuyến bay hoặc nhân viên hỗ trợ. |
| Correction path | Chưa thấy có nút sửa thông tin, quay lại, chọn lại intent, hoặc handoff kèm context. | User có thể sửa mã chuyến bay/ngày bay/mã đặt chỗ bằng button, xem lịch sử thông tin đã nhập, và chuyển nhân viên với context đã gom sẵn. |

## 5. Finding viết thành product decision

Khi user hỏi các tác vụ có tính hành động cao như đổi vé, tra delay hoặc mang hành lý đặc biệt, NEO nhận diện đúng intent nhưng thường dừng ở mức hỏi thêm thông tin hoặc trả policy dài.

Hậu quả là user chưa hoàn thành được task trong chat: chưa biết phí đổi vé, chưa biết delay bao lâu ngay lúc đang ở sân bay, hoặc phải tự rút ra kết luận từ câu trả lời dài.

Lỗi thuộc các layer:

- **Promise layer:** NEO được cảm nhận như trợ lý xử lý nghiệp vụ, nhưng nhiều luồng vẫn vận hành như FAQ/routing.
- **Intent layer:** Nhận diện ý định chính tốt, nhưng chưa khai thác hết context khẩn cấp như "đang ở sân bay" hoặc nhu cầu "mất bao nhiêu tiền".
- **Data-tool layer:** Có dấu hiệu cần booking tool, flight status tool và knowledge base, nhưng kết quả chưa được đóng gói thành action hoàn chỉnh.
- **Safety layer:** Tốt. Bot không bịa dữ liệu khi thiếu thông tin.
- **UX recovery layer:** Yếu nhất. Thiếu correction button, suggested actions, handoff và flow có cấu trúc.

Product decision đề xuất: thêm **Task Completion Flow** cho NEO, ưu tiên 3 nhóm intent có tần suất cao: đổi vé, tình trạng chuyến bay và hành lý đặc biệt.

## 6. Sketch As-is / To-be

| As-is flow hiện tại | To-be flow đề xuất |
|---|---|
| 1. User hỏi: "Đổi vé VN257 ngày mai mất bao nhiêu tiền?" | 1. User hỏi cùng câu trên. |
| 2. NEO nhận diện cần thêm mã đặt chỗ. | 2. NEO xác định intent đổi vé và mở mini form. |
| 3. NEO yêu cầu user nhập mã đặt chỗ bằng chat. | 3. User nhập mã đặt chỗ, chọn đổi ngày/đổi giờ/đổi hành trình. |
| 4. User phải tự tiếp tục hỏi và không thấy trạng thái xử lý. | 4. NEO gọi pricing/booking tool, trả phí dự kiến, điều kiện vé và nút tiếp tục đổi vé. |

| As-is flow hiện tại | To-be flow đề xuất |
|---|---|
| 1. User nói đang ở sân bay và hỏi VN154 delay bao lâu. | 1. User hỏi cùng câu trên. |
| 2. NEO xác nhận lại chuyến bay/ngày bay. | 2. NEO suy luận ngày hiện tại nếu không mơ hồ, trả ngay tình trạng chuyến bay. |
| 3. User chưa có câu trả lời delay. | 3. NEO hiện delay bao lâu, giờ khởi hành mới, gate, boarding và thời điểm cập nhật dữ liệu. |
| 4. Nếu thiếu dữ liệu, user phải tiếp tục chat. | 4. Nếu không có dữ liệu, NEO đưa nút liên hệ nhân viên sân bay/CSKH kèm context. |

| As-is flow hiện tại | To-be flow đề xuất |
|---|---|
| 1. User hỏi có mang xe đạp carbon được không. | 1. User hỏi cùng câu trên. |
| 2. NEO trả lời policy dài. | 2. NEO trả lời trước bằng kết luận: "Có, nhưng phải ký gửi và đóng gói đúng quy định." |
| 3. User phải tự đọc để biết cần làm gì. | 3. NEO tóm tắt 3 việc cần làm: đóng gói, đăng ký trước, kiểm tra phí/kích thước. |
| 4. Chưa có action tiếp theo. | 4. NEO hiện nút đăng ký hành lý đặc biệt, xem bảng phí, liên hệ hỗ trợ. |

**Điểm gãy lớn nhất trong As-is:** NEO hỏi thêm hoặc trả lời policy, nhưng không đưa user vào một workflow rõ ràng để kết thúc task.

**Điểm sửa chính trong To-be:** Chuyển NEO từ mô hình `Information Retrieval` sang `Task Completion`: hỏi đúng dữ liệu còn thiếu, gọi tool phù hợp, trả kết quả có thể hành động, và luôn có recovery path.

## 7. SPEC change đề xuất

### REQ-NEO-TASK-COMPLETION-FLOW

Khi user hỏi về đổi vé, tình trạng chuyến bay hoặc hành lý đặc biệt, NEO không được chỉ trả lời bằng text chung nếu tác vụ có thể chuyển thành workflow. NEO phải kích hoạt task completion flow có dữ liệu, action, fallback và handoff.

Yêu cầu chi tiết:

1. Nhận diện intent thuộc nhóm đổi vé, flight status, hành lý đặc biệt.
2. Phân loại mức độ khẩn cấp dựa trên context như "đang ở sân bay", "ngày mai", "delay", "mất bao nhiêu tiền".
3. Nếu thiếu dữ liệu, hỏi lại bằng form/options thay vì chỉ chat tự do.
4. Với đổi vé, yêu cầu mã đặt chỗ và loại thay đổi, sau đó trả phí dự kiến/điều kiện vé nếu tool cho phép.
5. Với flight status, ưu tiên trả kết quả realtime gồm delay, giờ mới, gate, boarding và thời điểm cập nhật.
6. Với hành lý đặc biệt, trả kết luận ngắn gọn trước, sau đó mới đưa quy định chi tiết.
7. Luôn có suggested actions phù hợp: tiếp tục đổi vé, xem bảng phí, đăng ký hành lý, liên hệ CSKH.
8. Cho phép user sửa thông tin đã nhập như mã chuyến bay, ngày bay, mã đặt chỗ.
9. Nếu AI không đủ dữ liệu hoặc tool lỗi, hiện fallback rõ ràng và handoff sang nhân viên với context hội thoại.
10. Hiển thị nguồn dữ liệu hoặc thời điểm cập nhật với các thông tin nhạy cảm như delay, gate, phí đổi vé.

## 8. Test cases

| Test case | Input / tình huống | Expected behavior |
|---|---|---|
| TC01 - Đổi vé thiếu mã đặt chỗ | User hỏi: "Đổi vé VN257 ngày mai từ Hà Nội đi TP.HCM mất bao nhiêu tiền?" | NEO nhận diện đổi vé, giải thích cần mã đặt chỗ, hiện form nhập mã và options đổi ngày/đổi giờ/đổi hành trình. |
| TC02 - Đổi vé có mã đặt chỗ | User cung cấp mã đặt chỗ hợp lệ. | NEO trả phí dự kiến, điều kiện vé, hạn mức hành lý/chênh lệch giá nếu có và nút tiếp tục đổi vé. |
| TC03 - Delay tại sân bay | User hỏi: "Tôi đang ở sân bay Nội Bài, chuyến VN154 bị delay bao lâu?" | NEO trả tình trạng chuyến bay hiện tại nếu dữ liệu có sẵn: delay bao lâu, giờ khởi hành mới, gate, boarding, thời điểm cập nhật. |
| TC04 - Flight status mơ hồ | Có nhiều chuyến VN154 hoặc thiếu ngày bay. | NEO hỏi lại bằng options ngắn, không bắt user tự gõ lại toàn bộ câu hỏi. |
| TC05 - Hành lý đặc biệt | User hỏi: "Tôi mang xe đạp thể thao carbon lên máy bay được không?" | NEO trả kết luận ngắn gọn "Có, dưới dạng hành lý ký gửi", sau đó tóm tắt đóng gói, đăng ký trước, giới hạn và phí. |
| TC06 - Tool không có dữ liệu | Flight status/booking tool không trả kết quả. | NEO nói rõ không trả được vì thiếu dữ liệu, đưa fallback/handoff và giữ context để nhân viên tiếp tục xử lý. |

## 9. Kết luận

Finding quan trọng nhất: NEO không yếu ở việc hiểu intent. Điểm cần cải thiện là khả năng biến intent thành workflow hoàn tất. Trong 3 query thử nghiệm, bot an toàn và nhận diện đúng chủ đề, nhưng user vẫn có nguy cơ bị kẹt vì thiếu action tiếp theo.

SPEC nên đổi theo hướng thêm **Task Completion Flow** cho các tác vụ hàng không phổ biến. NEO cần hỏi ít hơn, hành động nhiều hơn, trả kết quả ngắn gọn theo ngữ cảnh và luôn có recovery path khi thiếu dữ liệu hoặc khi user cần sửa thông tin.
