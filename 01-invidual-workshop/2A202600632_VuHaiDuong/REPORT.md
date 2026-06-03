# Report App Teardown: MoMo Moni

WebHTML Analysis: https://global-green-sjkdeiv1.edgeone.dev/

## 0. Thông tin nhanh

**Sản phẩm:** MoMo - Moni  
**AI feature:** Trợ thủ tài chính / chatbot phân tích chi tiêu  
**Case dùng thử:** User hỏi về "chi tiêu linh tinh của tôi ở trong MoMo"  
**Input quan sát:** "Cho tôi biết chi tiêu linh tinh của tôi ở trong MoMo."  
**Evidence:** Screenshot trong `anh_hoidap.jpg` và nội dung phân tích trong `main.html`

Mục tiêu của teardown này không phải đánh giá câu trả lời có hay hay không, mà là tìm điểm gãy trong workflow thật: user hỏi về dữ liệu chi tiêu cá nhân, nhưng AI chưa chuyển được câu hỏi đó thành một luồng hành động có dữ liệu, sửa sai và ghi nhớ.

## 1. Product được chọn

| Hạng mục | Nội dung |
|---|---|
| Product | MoMo |
| AI feature | Moni - trợ thủ tài chính, chatbot hỗ trợ phân tích chi tiêu |
| User chính trong case | Người dùng MoMo muốn hiểu hoặc sửa nhóm chi tiêu cá nhân |
| Task kỳ vọng | Hỏi bằng ngôn ngữ tự nhiên để xem, hiểu, sửa hoặc cá nhân hóa nhóm "Linh tinh" |

## 2. Promise vs Reality

### Promise

Promise ngầm của Moni là người dùng có thể hỏi trợ thủ tài chính bằng ngôn ngữ tự nhiên và được hỗ trợ trên dữ liệu chi tiêu thật trong MoMo. Với câu hỏi có cụm "của tôi", user kỳ vọng AI hiểu rằng đây là dữ liệu cá nhân, không phải một câu hỏi định nghĩa chung.

User có thể kỳ vọng Moni làm được các việc sau:

- Cho biết các giao dịch nào đang được xếp vào nhóm "Linh tinh".
- Giải thích vì sao các khoản đó bị xếp vào nhóm này.
- Cho phép đổi phân loại nếu hệ thống phân loại sai.
- Ghi nhớ rule cá nhân cho các giao dịch tương tự trong tương lai.

### Reality

Trong evidence, Moni dừng lại ở mức trả lời chung: nói chưa có tính năng trực tiếp và hướng user tự kiểm tra thủ công trong app. AI có nhận ra phần nào ngữ cảnh chi tiêu, nhưng chưa nối được câu hỏi sang workflow có hành động.

Điểm gãy chính:

- Không hiển thị danh sách giao dịch liên quan.
- Không nói rõ source dữ liệu đang dựa vào đâu.
- Không hỏi lại khi intent còn mơ hồ.
- Không có nút sửa phân loại.
- Không có undo.
- Không có correction log hoặc rule learning.

### Impact

User vẫn phải tự đi tìm trong app, tự lọc giao dịch, tự đoán tiêu chí phân loại và tự sửa nếu có lỗi. Như vậy, task chính của user chưa được hoàn thành dù AI đã trả lời.

## 3. Evidence quan sát được

**Prompt/input đã thử:**  
"Cho tôi biết chi tiêu linh tinh của tôi ở trong MoMo."

**Hành vi quan sát được:**  
Moni trả lời theo hướng giải thích hoặc hướng dẫn chung, nói chưa có tính năng trực tiếp để xem phần này trong chat và gợi ý user tự vào phần quản lý chi tiêu để kiểm tra.

**Vì sao evidence này quan trọng:**  
Cụm "của tôi" là tín hiệu rất mạnh cho thấy user muốn truy vấn dữ liệu cá nhân. Nếu AI chỉ trả lời như một câu hỏi khái niệm, sản phẩm bị gãy ở tầng intent và tầng recovery UX.

## 4. Bốn paths

| Path | Hiện trạng quan sát | Kỳ vọng tốt hơn |
|---|---|---|
| Happy path | Moni có thể hiểu user hỏi về chi tiêu, nhưng chưa đưa ra dữ liệu hoặc action cụ thể. | Moni nhận diện đây là câu hỏi về phân loại chi tiêu cá nhân, hiển thị giao dịch liên quan hoặc hỏi user muốn xem phần nào. |
| Low-confidence path | Chưa thấy bước hỏi lại. Moni có vẻ chọn hướng trả lời chung. | Khi intent mơ hồ, Moni hỏi lại bằng tối đa 3 lựa chọn: xem giao dịch, hiểu tiêu chí, đổi phân loại/tạo rule. |
| Failure path | Moni nói chưa có tính năng trực tiếp và hướng user tự kiểm tra. | Nếu không đủ quyền truy cập dữ liệu trong chat, Moni cần mở deep link sang Quản lý chi tiêu, giao dịch đã lọc hoặc CSKH. |
| Correction path | Chưa có cơ chế sửa phân loại, undo, log hoặc học lại rule. | User có thể đổi tag giao dịch ngay trong flow, xác nhận thay đổi, áp dụng rule cho lần sau và hoàn tác nếu sửa nhầm. |

## 5. Finding viết thành product decision

Khi user hỏi "Cho tôi biết chi tiêu linh tinh của tôi ở trong MoMo", AI hiểu một phần chủ đề nhưng xử lý như một câu hỏi hướng dẫn chung thay vì nhận ra intent mơ hồ về dữ liệu chi tiêu cá nhân.

Hậu quả là user không thấy giao dịch nào đang thuộc nhóm "Linh tinh", không hiểu tiêu chí phân loại, không sửa được phân loại sai và vẫn phải tự đi tìm trong app.

Lỗi thuộc các layer:

- **Promise layer:** Trợ thủ tài chính tạo kỳ vọng hỗ trợ trên dữ liệu thật, nhưng phản hồi chưa thực hiện được hành động tài chính cá nhân.
- **Intent layer:** AI chưa phân biệt rõ giữa hỏi định nghĩa chung và hỏi dữ liệu "của tôi".
- **Data-tool layer:** Chat layer chưa show được transaction data, source hoặc action trên giao dịch.
- **UX recovery layer:** Không có re-ask, option, handoff đủ rõ.
- **Correction layer:** Không có correction log, rule learning hoặc undo.

Product decision đề xuất: thêm **Classification Recovery Flow** cho các câu hỏi liên quan đến danh mục chi tiêu mơ hồ hoặc nghi ngờ phân loại sai.

## 6. Sketch As-is / To-be

| As-is flow hiện tại | To-be flow đề xuất |
|---|---|
| 1. User hỏi: "Cho tôi biết chi tiêu linh tinh của tôi ở trong MoMo." | 1. User hỏi cùng câu trên. |
| 2. Moni nhận diện một phần chủ đề "chi tiêu linh tinh". | 2. Moni nhận diện đây là intent mơ hồ về classification recovery. |
| 3. Moni trả lời bằng text chung và nói chưa có tính năng trực tiếp. | 3. Moni hỏi lại bằng option: xem giao dịch, hiểu tiêu chí, đổi phân loại/tạo rule. |
| 4. User được hướng dẫn tự vào Quản lý chi tiêu. | 4. Nếu user chọn xem giao dịch, Moni show source: ví dụ giao dịch 30 ngày gần đây, merchant, số tiền, tag hiện tại. |
| 5. User tự lọc, tự hiểu, tự sửa nếu tìm được. | 5. User có thể đổi tag ngay trong flow, ví dụ Circle K 35.000đ từ "Linh tinh" sang "Ăn uống". |
| 6. Không có log, không có undo, không có rule learning. | 6. Moni xác nhận thay đổi, hỏi có áp dụng rule cho giao dịch tương tự không, lưu correction log và cho phép undo. |

**Điểm gãy lớn nhất trong As-is:** Sau khi Moni nói chưa có tính năng trực tiếp, user không còn một recovery path rõ ràng để hoàn thành task.

**Điểm sửa chính trong To-be:** Chuyển câu trả lời AI từ text hướng dẫn sang workflow có dữ liệu, lựa chọn, sửa sai, undo và ghi nhớ.

## 7. SPEC change đề xuất

### REQ-MONI-CLASSIFICATION-RECOVERY

Khi user hỏi về danh mục chi tiêu mơ hồ hoặc nghi ngờ phân loại sai, Moni không được chỉ trả lời bằng text giải thích chung. Moni phải kích hoạt recovery flow có dữ liệu, action, handoff, undo và correction log.

Yêu cầu chi tiết:

1. Nhận diện intent thuộc nhóm classification recovery khi user hỏi về nhóm chi tiêu như "Linh tinh", "Ăn uống", "Mua sắm", "Đi lại" kèm tín hiệu cá nhân như "của tôi".
2. Nếu intent chưa rõ, hỏi lại user bằng tối đa 3 lựa chọn ngắn.
3. Hiển thị source dữ liệu mà AI đang dựa vào, ví dụ giao dịch 30 ngày gần đây.
4. Cho phép xem danh sách giao dịch liên quan.
5. Cho phép đổi phân loại giao dịch ngay trong flow.
6. Cho phép tạo hoặc áp dụng rule cho giao dịch tương tự.
7. Hiển thị xác nhận sau khi sửa.
8. Cho phép undo.
9. Lưu correction log để user biết hệ thống đã đổi gì và khi nào.
10. Nếu AI không đủ dữ liệu, handoff sang Quản lý chi tiêu, màn hình giao dịch liên quan hoặc CSKH.

## 8. Test cases

| Test case | Input / tình huống | Expected behavior |
|---|---|---|
| TC01 - Intent mơ hồ | User hỏi: "Cho tôi biết chi tiêu linh tinh của tôi ở trong MoMo." | Moni không trả lời chung ngay. Moni hỏi lại bằng option: xem giao dịch, hiểu tiêu chí, đổi phân loại/tạo rule. |
| TC02 - Xem giao dịch | User chọn "Xem giao dịch đang được xếp là Linh tinh". | Moni hiển thị danh sách giao dịch liên quan, source dữ liệu và action trên từng giao dịch. |
| TC03 - Sửa phân loại | User đổi Circle K 35.000đ từ "Linh tinh" sang "Ăn uống". | Moni xác nhận thay đổi, lưu correction log, hỏi có áp dụng rule cho giao dịch tương tự không và cho phép undo. |
| TC04 - AI không đủ dữ liệu | Moni không thể truy cập chi tiết giao dịch trong chat. | Moni nói rõ giới hạn và đưa button mở Quản lý chi tiêu, mở giao dịch liên quan hoặc liên hệ CSKH. |

## 9. Kết luận

Finding quan trọng nhất: vấn đề không nằm ở việc Moni cần viết câu trả lời dài hơn. Vấn đề là Moni cần chuyển từ conversation sang workflow. Với case "chi tiêu linh tinh của tôi", sản phẩm cần cho user xem dữ liệu, hiểu tiêu chí, sửa phân loại, hoàn tác và dạy lại hệ thống bằng correction log/rule learning.

SPEC nên đổi theo hướng thêm **Classification Recovery Flow** cho các tình huống user hỏi về nhóm chi tiêu mơ hồ hoặc nghi ngờ hệ thống phân loại sai.
