## Promise
- "Đóng vai trò như một "người quản gia" tài chính cá nhân, giúp người dùng quản lý chi tiêu hiệu quả, đưa ra các quyết định tài chính thông minh và tối ưu hóa trải nghiệm trên nền tảng."
- "Cá nhân hóa và đơn giản hóa việc quản lý tài chính"
- "Moni tự tạo ra các câu trả lời linh hoạt và phù hợp với ngữ cảnh, thay vì chỉ phản hồi theo các kịch bản lập trình sẵn, mang lại một cuộc trò chuyện thực thụ."
- "Moni không hoạt động độc lập mà được tích hợp sâu, có thể "hiểu" được các hành động của người dùng trên MoMo, từ thanh toán hóa đơn đến chuyển tiền, để đưa ra gợi ý và phân tích phù hợp với từng ngữ cảnh."

--> infered promise: có thể phân loại các chi tiêu, có thể tổng hợp các chi tiêu theo nhóm, có thể gợi ý nên giảm chi tiêu gì.

---

## Notes:
- Moni không xuất hiện default, phải search hoặc lướt rất xa.
- Với câu trả lời có thể trích xuất (6 tháng vừa qua tiêu bao nhiêu tiền 1.jpg, ước lượng chi tiêu linh tinh 2.jpg), trả lời oke; tiếp tục hỏi sâu hơn sẽ đi sâu hơn được, data persist.
- Ở error path (khi hỏi nhóm nào chi tiêu nhiều nhất 3.jpg), app không thể làm được, display message không làm được.
- Ở wrong path (khi hỏi nhóm nào 4.jpg), trả về tổng tiêu, không phân loại nhóm. khi feedback (có nút câu trả lời hữu ích) rằng không liên quan (5.jpg), không acknowlege, lặp lại câu trả lời sai cũ.

---

## SPEC change đề xuất

Finding này nên đổi SPEC của Moni theo hướng:

> Moni không chỉ trả lời tổng chi tiêu, mà phải hỗ trợ **expense insight workflow**: tổng hợp → phân nhóm → xếp hạng → drill down → correction.

### Requirement cụ thể

| Requirement                  | Mô tả                                                                                                        |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------ |
| R1 — Group expense analysis  | Moni phải trả lời được top nhóm chi tiêu theo khoảng thời gian                                               |
| R2 — Drill-down              | User có thể hỏi tiếp “trong nhóm đó gồm khoản nào?”                                                          |
| R3 — Low-confidence fallback | Nếu nhóm chưa rõ, Moni phải hỏi lại hoặc đề xuất cách phân loại                                              |
| R4 — Correction handling     | Nếu user nói câu trả lời sai/không liên quan, Moni phải xác định lỗi intent và thử lại                       |
| R5 — Boundary response       | Với câu ngoài phạm vi như vay nhanh, Moni nên nói rõ giới hạn nhưng vẫn gợi ý sản phẩm MoMo liên quan nếu có |

---

## Test cases nên thêm vào SPEC

| Test case                                            | Expected behavior                                                                             |
| ---------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| “6 tháng qua tôi tiêu bao nhiêu?”                    | Trả tổng chi tiêu, số giao dịch, trung bình/ngày                                              |
| “Nhóm nào chi nhiều nhất?”                           | Trả top nhóm chi tiêu, số tiền, % tổng chi                                                    |
| “Liệt kê top 3 nhóm chi tiêu lớn nhất”               | Trả bảng top 3 nhóm                                                                           |
| “Trong nhóm linh tinh gồm khoản nào?”                | Drill down danh sách giao dịch thuộc nhóm đó                                                  |
| “Câu trả lời không liên quan, tôi hỏi nhóm chi tiêu” | Acknowledge lỗi, re-run theo intent nhóm chi tiêu                                             |
| “Nếu tôi muốn vay nhanh nên vay khoản nào?”          | Nói rõ phạm vi hỗ trợ, chỉ gợi ý nếu có sản phẩm MoMo phù hợp, tránh tư vấn tài chính quá mức |

---

## Kết luận teardown

Moni có nền tảng tốt ở các câu hỏi tổng hợp đơn giản vì có thể truy xuất dữ liệu giao dịch và trả lời bằng số liệu cụ thể. Tuy nhiên, với promise là “quản gia tài chính cá nhân”, use case quan trọng nhất không chỉ là “tôi đã tiêu bao nhiêu”, mà là **“tôi đang tiêu nhiều nhất vào đâu và nên làm gì tiếp theo”**.

Điểm gãy lớn nhất là Moni chưa xử lý ổn định workflow phân tích nhóm chi tiêu. Vì vậy, product decision nên là: ưu tiên xây dựng **expense category insight flow** trước khi mở rộng sang các tư vấn tài chính phức tạp hơn.