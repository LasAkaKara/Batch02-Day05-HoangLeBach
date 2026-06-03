Dưới đây là bản **app teardown phát triển từ note + template**, dựa trên evidence bạn đã test Moni.

---

# App Teardown — MoMo Moni

## 1. Product được chọn

| Mục           | Nội dung                                                                                              |
| ------------- | ----------------------------------------------------------------------------------------------------- |
| Product       | MoMo                                                                                                  |
| AI feature    | Moni — trợ lý tài chính cá nhân / chatbot phân tích chi tiêu                                          |
| Cách truy cập | Trong app MoMo, nhưng không xuất hiện mặc định; người dùng phải search hoặc lướt khá xa mới thấy      |
| User chính    | Người dùng MoMo có nhiều giao dịch, muốn hiểu mình tiêu gì, tiêu bao nhiêu, nhóm nào đang “ngốn tiền” |

---

## 2. Promise vs Reality

### Promise

Theo note, Moni được mô tả như một “người quản gia” tài chính cá nhân, giúp người dùng quản lý chi tiêu, đưa ra quyết định tài chính thông minh, cá nhân hóa trải nghiệm và hiểu được các hành động của người dùng trên MoMo như thanh toán, chuyển tiền, hóa đơn. Từ đó có thể infer rằng người dùng kỳ vọng Moni có khả năng tổng hợp chi tiêu, phân loại nhóm chi tiêu, và đưa ra insight hoặc gợi ý giảm chi. 

### Kỳ vọng của user khi dùng thật

User không chỉ hỏi tổng số tiền đã tiêu, mà muốn đi sâu hơn:

* 6 tháng qua tôi tiêu bao nhiêu?
* Nhóm chi tiêu nào lớn nhất?
* Tỉ lệ chi tiêu linh tinh là bao nhiêu?
* Nếu muốn vay nhanh thì nên vay khoản nào?

Với promise “quản gia tài chính cá nhân”, user có lý do để kỳ vọng Moni không chỉ trả lời số tổng, mà còn phân tích theo nhóm, giải thích, và đề xuất bước tiếp theo.

### Reality quan sát được

Moni làm khá tốt ở các câu hỏi có thể trích xuất trực tiếp từ dữ liệu giao dịch, ví dụ:

* “6 tháng vừa qua tôi đã tiêu bao nhiêu tiền” → trả về tổng chi tiêu **5.389.400đ**, **29 giao dịch**, trung bình **29.450đ/ngày**.
* “Ước lượng tỉ lệ chi tiêu linh tinh của tôi” → trả về nhóm **Chi phí phát sinh (linh tinh)** là **131.638đ**, **5 giao dịch**, chiếm khoảng **2.4%** tổng chi tiêu.

Nhưng Moni gãy khi user hỏi câu có intent phân tích hơn:

* “Liệt kê các nhóm chi tiêu lớn nhất, mỗi nhóm bao nhiêu” → Moni chỉ lặp lại tổng chi tiêu, không liệt kê nhóm.
* Khi user feedback “câu trả lời không liên quan, không liệt kê được nhóm”, Moni xin lỗi nhưng vẫn không sửa được lỗi chính.
* Khi hỏi “Nhóm nào chi nhiều nhất?”, Moni nói cần xem chi tiết từng nhóm để xác định, nhưng hiện tại hệ thống chỉ trả về tổng số tiền và số giao dịch.

Điểm gãy chính: **Moni có dữ liệu giao dịch, có thể trả lời một số nhóm cụ thể, nhưng không xử lý ổn định task phân tích top nhóm chi tiêu.**

---

## 3. Evidence table

| Evidence       | Prompt của user                                          | Output quan sát được                                      | Nhận xét                                           |
| -------------- | -------------------------------------------------------- | --------------------------------------------------------- | -------------------------------------------------- |
| Screenshot 1   | “6 tháng vừa qua tôi đã tiêu bao nhiêu tiền”             | Trả về tổng chi tiêu, số giao dịch, trung bình/ngày       | Happy path tốt: câu hỏi tổng hợp đơn giản          |
| Screenshot 2   | “ước lượng tỉ lệ chi tiêu linh tinh của tôi”             | Trả về nhóm “Chi phí phát sinh (linh tinh)” và tỉ lệ 2.4% | Có khả năng truy xuất nhóm cụ thể                  |
| Screenshot 3/4 | “liệt kê các nhóm chi tiêu lớn nhất, mỗi nhóm bao nhiêu” | Trả về tổng chi tiêu, không liệt kê nhóm                  | Wrong path: hiểu sai hoặc thiếu tool phân nhóm     |
| Screenshot 5   | User feedback “câu trả lời không liên quan…”             | Xin lỗi nhưng tiếp tục yêu cầu xác nhận xem chi tiết      | Correction không sửa được vấn đề                   |
| Screenshot 6   | “nếu tôi muốn vay nhanh…”                                | Trả lời ngoài phạm vi: chỉ hỗ trợ sản phẩm MoMo           | Safety / boundary path tốt hơn, nhưng chưa helpful |

---

## 4. Four paths

Template yêu cầu phân tích 4 path: Happy, Low-confidence, Failure, Correction. 

### Path 1 — Happy path

**Trigger:** User hỏi câu đơn giản, có thể query trực tiếp từ dữ liệu giao dịch.

**Ví dụ:**
“6 tháng vừa qua tôi đã tiêu bao nhiêu tiền”

**Moni làm gì:**
Trả về:

* Tổng chi tiêu: **5.389.400đ**
* Số giao dịch: **29**
* Trung bình mỗi ngày: **29.450đ**

**Đánh giá:**
Happy path hoạt động tốt vì intent rõ, data aggregation đơn giản, không cần phân loại sâu. User nhận được câu trả lời nhanh, có số liệu cụ thể, đúng với promise “đơn giản hóa quản lý tài chính”.

**Product implication:**
Giữ flow này, nhưng nên thêm CTA tiếp theo như:

> “Bạn muốn xem top 3 nhóm chi tiêu lớn nhất không?”

---

### Path 2 — Low-confidence path

**Trigger:** User hỏi câu cần phân loại hoặc xếp hạng nhóm chi tiêu.

**Ví dụ:**
“Nhóm nào chi nhiều nhất?”

**Moni làm gì hiện tại:**
Moni không hỏi lại để clarify, cũng không show options. Thay vào đó, Moni nói rằng cần xem chi tiết từng nhóm để xác định, nhưng hiện tại hệ thống chỉ trả về tổng số tiền và số giao dịch.

**Vấn đề:**
Đây là tình huống low-confidence nhưng Moni chưa có low-confidence UX thật sự. Thay vì nói “tôi chưa có đủ dữ liệu”, hệ thống nên:

* Nêu rõ đang thiếu gì.
* Đưa lựa chọn cho user.
* Đề xuất cách tiếp tục.

**To-be suggestion:**
Khi không chắc nhóm chi tiêu nào là đúng, Moni nên hỏi:

> “Mình có thể phân nhóm theo danh mục MoMo hiện có như Ăn uống, Di chuyển, Hóa đơn, Mua sắm, Chi phí phát sinh. Bạn muốn xem top 3 nhóm trong 6 tháng không?”

Hoặc nếu dữ liệu chưa đủ:

> “Hiện một số giao dịch chưa được gắn nhóm. Mình có thể ước lượng top nhóm dựa trên tên giao dịch, hoặc chỉ hiển thị các giao dịch đã được phân loại. Bạn chọn cách nào?”

---

### Path 3 — Failure path

**Trigger:** User yêu cầu liệt kê nhóm chi tiêu lớn nhất.

**Ví dụ:**
“Liệt kê các nhóm chi tiêu lớn nhất, mỗi nhóm bao nhiêu”

**Moni làm gì:**
Moni trả lời tổng chi tiêu, số giao dịch, chi tiêu trung bình/ngày, nhưng không trả lời đúng câu hỏi “nhóm nào” và “mỗi nhóm bao nhiêu”.

**Failure:**
AI/product không fulfill đúng intent phân tích nhóm.

**Impact:**
User mất niềm tin vì Moni có vẻ hiểu dữ liệu tổng, nhưng không thể đi sâu vào insight quan trọng nhất. Với một trợ lý tài chính, “nhóm nào chi nhiều nhất” là use case cốt lõi, không phải edge case.

**Layer lỗi:**

* Intent understanding: nhận ra câu hỏi là “rank expense categories” chưa ổn định.
* Data-tool: có thể chưa có tool trả về group-by category.
* UX recovery: khi không làm được, không đưa ra fallback hữu ích.
* Promise gap: promise “quản gia tài chính” cao hơn khả năng thực tế.

---

### Path 4 — Correction path

**Trigger:** User feedback rằng câu trả lời không liên quan.

**Ví dụ:**
“Câu trả lời không liên quan, không liệt kê được nhóm”

**Moni làm gì:**
Moni xin lỗi, nhưng vẫn không sửa được lỗi. Câu trả lời tiếp tục xoay quanh tổng chi tiêu và yêu cầu user xác nhận xem chi tiết.

**Vấn đề:**
Correction không được dùng như signal để sửa intent. User đã nói rõ lỗi là “không liệt kê được nhóm”, nhưng Moni không chuyển sang flow phân nhóm hoặc fallback.

**Impact:**
Feedback button và text correction trở nên yếu, vì user cảm giác hệ thống không học hoặc không hiểu correction.

**To-be suggestion:**
Sau khi user sửa, Moni nên acknowledge đúng lỗi:

> “Bạn nói đúng, câu trước mình chỉ trả tổng chi tiêu chứ chưa liệt kê theo nhóm. Mình sẽ thử phân nhóm lại theo danh mục giao dịch.”

Sau đó có 2 trường hợp:

* Nếu có data group-by: hiển thị top nhóm.
* Nếu chưa có data group-by: nói rõ giới hạn và show fallback, ví dụ top merchant, top loại giao dịch, hoặc danh sách giao dịch chưa phân loại.

---

## 5. Core finding viết thành product decision

### Finding 1 — Moni trả lời tốt các câu hỏi tổng hợp, nhưng gãy ở insight theo nhóm

Khi user hỏi **“nhóm chi tiêu nào lớn nhất / mỗi nhóm bao nhiêu”**, Moni trả về **tổng chi tiêu** thay vì phân tích theo nhóm, hậu quả là user không nhận được insight tài chính quan trọng để quyết định nên giảm chi ở đâu. Lỗi thuộc layer **intent + data-tool + promise gap**. Nên sửa bằng requirement: Moni phải có tool `group_expenses_by_category(time_range)` và test case riêng cho các câu hỏi top nhóm chi tiêu.

---

### Finding 2 — Low-confidence path chưa đủ rõ

Khi Moni không chắc hoặc không thể phân loại nhóm chi tiêu, sản phẩm hiện không hỏi lại, không show option, và không giải thích thiếu dữ liệu gì. Hậu quả là user không biết phải hỏi lại thế nào để đi tiếp. Lỗi thuộc layer **UX recovery**. Nên sửa bằng low-confidence UX: hỏi user muốn phân nhóm theo danh mục, merchant, loại giao dịch, hay chỉ xem giao dịch đã được phân loại.

---

### Finding 3 — Correction không được dùng để recover intent

Khi user feedback rằng câu trả lời không liên quan, Moni xin lỗi nhưng không sửa đúng vấn đề. Hậu quả là feedback loop bị “giả”, user không thấy correction có tác dụng. Lỗi thuộc layer **correction handling + conversation memory**. Nên sửa bằng requirement: nếu user correction chứa intent rõ như “không liệt kê được nhóm”, Moni phải re-run task với intent đã sửa hoặc giải thích rõ vì sao không thể.

---

## 6. As-is / To-be sketch

| As-is flow hiện tại                                         | To-be flow đề xuất                                                                                          |
| ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| User hỏi: “6 tháng vừa qua tôi tiêu bao nhiêu?”             | User hỏi: “6 tháng vừa qua tôi tiêu bao nhiêu?”                                                             |
| Moni trả tổng chi tiêu, số giao dịch, trung bình/ngày       | Moni trả tổng chi tiêu, số giao dịch, trung bình/ngày                                                       |
| User hỏi tiếp: “Nhóm nào chi nhiều nhất?”                   | Moni chủ động gợi ý: “Bạn muốn xem top 3 nhóm chi tiêu không?”                                              |
| Moni trả lại tổng chi tiêu hoặc nói chưa xác định được nhóm | User chọn xem top nhóm                                                                                      |
| User feedback: “Không liên quan, không liệt kê nhóm”        | Moni gọi tool phân nhóm: `group by category`                                                                |
| Moni xin lỗi nhưng chưa sửa được                            | Moni hiển thị top nhóm + số tiền + số giao dịch                                                             |
| User phải tự hỏi lại hoặc bỏ cuộc                           | Nếu thiếu data, Moni hỏi: “Bạn muốn mình ước lượng theo tên giao dịch hay chỉ dùng giao dịch đã phân loại?” |

---


- [x] Có ít nhất 1 screenshot hoặc observation cụ thể.
- [x] Có đủ 4 paths hoặc nói rõ path nào chưa có trong product.
- [x] Finding được viết thành product decision, không chỉ là nhận xét.
- [x] Sketch có as-is và to-be.
- [x] Có một câu nói rõ finding này sẽ đổi gì trong SPEC.
