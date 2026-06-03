## 1. Nhóm và track

**Tên nhóm:**  Chill Guys
**Track:**  Travel and Hospitality
**Product/app đã chọn:**  Trip.com
**Build slice đang nghĩ:**  Khách du lịch muốn tìm kiếm phương tiện di chuyển -> AI hỏi 3 câu -> Gợi ý nhiều phương án hợp lý -> Failure: flag lại, chuyển hướng người dùng tự kiếm

## 2. Self-use evidence

Nhóm tự dùng Trip.com chatbot với nhu cầu chính: khách du lịch muốn tìm kiếm phương tiện di chuyển. Nhóm thử nhiều câu hỏi tự nhiên bằng tiếng Việt, bao gồm tìm vé máy bay, vé tàu, thuê xe, so sánh lựa chọn di chuyển và yêu cầu chatbot đưa ra gợi ý cụ thể.

| Observation | Screenshot/link | Path liên quan | Điều học được |
|---|---|---|---|
| Khi người dùng hỏi một nhu cầu rất phổ biến như “Tôi muốn tìm vé từ Hà Nội đi Phú Quốc cuối tuần này”, chatbot không hỏi thêm thông tin cần thiết như ngày đi, ngày về, số người, ngân sách, ưu tiên giờ bay. Thay vào đó, chatbot chỉ hướng người dùng sang trang “Đặt vé máy bay” và đưa hướng dẫn chung. | <a href="Image/Uncertain_Path.png"><img src="Image/Uncertain_Path.png" alt="Uncertain_Path" width="160"></a> | `Low-confidence` | Chatbot chưa thực sự đóng vai trò travel assistant. Nó không hiểu intent đủ sâu để chủ động thu thập thông tin và tạo hành trình tìm kiếm. Với use case tìm phương tiện di chuyển, đây là điểm gãy lớn vì người dùng vẫn phải tự thao tác gần như toàn bộ. |
| Với câu “Tìm cho tôi chuyến đi từ Sài Gòn ra Đà Nẵng ngày 10 tháng 6”, chatbot vẫn trả lời bằng template hướng dẫn đặt vé máy bay, không parse được điểm đi, điểm đến, ngày đi để tạo search query. | <a href="Image/Uncertain_Path_2.png"><img src="Image/Uncertain_Path_2.png" alt="Uncertain_Path_2" width="160"></a> | `Low-confidence` | Ngay cả khi người dùng đã cung cấp origin, destination và date, chatbot không chuyển thành action cụ thể. Điều này cho thấy hệ thống chưa có intent extraction + slot filling đủ tốt. |
| Khi người dùng hỏi “so sánh giá vé các hãng máy bay tại Việt Nam”, chatbot trả lời lạc đề về mã IATA và quy trình xác minh hãng hàng không. Đây không phải câu trả lời cho nhu cầu so sánh giá. | <a href="Image/Wrong_Answer_1.png"><img src="Image/Wrong_Answer_1.png" alt="Wrong_Answer_1" width="160"></a><br><a href="Image/Wrong_Answer_2.png"><img src="Image/Wrong_Answer_2.png" alt="Wrong_Answer_2" width="160"></a> | `Failure` | Chatbot có dấu hiệu hiểu sai keyword “hãng máy bay” thành thông tin về ngành hàng không / IATA. Nó không nhận ra người dùng đang hỏi về comparison shopping. Với travel planning, failure này nguy hiểm vì người dùng cần thông tin quyết định nhanh nhưng nhận được nội dung không liên quan. |
| Sau khi người dùng đánh giá chatbot tệ và ghi feedback “không hiểu câu trả hỏi, câu trả lời generic, không có search”, hệ thống chỉ chuyển sang human support chứ không có cơ chế tự sửa câu trả lời hoặc hỏi lại. | <a href="Image/Correction_1.png"><img src="Image/Correction_1.png" alt="Correction_1" width="160"></a><br><a href="Image/Correction_2.png"><img src="Image/Correction_2.png" alt="Correction_2" width="160"></a><br><a href="Image/Correction_3.png"><img src="Image/Correction_3.png" alt="Correction_3" width="160"></a> | `Correction` | Correction path hiện tại là `report / handoff` chứ chưa phải conversational correction. Chatbot không tận dụng feedback để thử lại, làm rõ intent, hoặc đề xuất query mới. |
| Trong flow vé tàu, khi người dùng hỏi “vậy bạn làm được điều gì”, chatbot không trả lời được năng lực của chính nó, chỉ báo không hiểu câu hỏi và gợi ý một câu hỏi khác. | <a href="Image/Can't_Understand_Question.png"><img src="Image/Can't_Understand_Question.png" alt="Can't_Understand_Question" width="160"></a> | `Failure` | Chatbot thiếu self-awareness về scope. Người dùng không biết nên hỏi gì, chatbot cũng không giải thích được nó có thể hỗ trợ gì. Đây là friction lớn ở bước onboarding hoặc khi người dùng gặp lỗi. |
| Với yêu cầu đặt vé tàu cụ thể “đặt cho tôi vé tàu ngày 3/6 đi Hà Nội từ Hải Phòng”, chatbot chỉ đưa link “Tìm kiếm tàu hoả” và hướng dẫn chung, không thực hiện tìm kiếm hoặc xác nhận thông tin. | <a href="Image/Can't_execute_request.png"><img src="Image/Can't_execute_request.png" alt="Can't_execute_request" width="160"></a> | `Failure` | Chatbot chưa execute được request, chỉ redirect. Với build slice của nhóm, đây là bằng chứng rõ rằng cần một agent có khả năng hỏi thêm thông tin, gọi search tool/API, rồi trả lại options. |
| Chatbot bị tách theo từng dịch vụ như vé máy bay, vé tàu, thuê xe. Người dùng phải chọn service trước, thay vì nói một nhu cầu tổng quát như “đi Hà Nội - Hải Phòng ngày 3/6 có option nào rẻ và đánh giá cao”. | <a href="Image/No_Unififed_Chatbot.png"><img src="Image/No_Unififed_Chatbot.png" alt="No_Unififed_Chatbot" width="160"></a> | `Failure` | Trip.com hiện chưa có unified mobility assistant. Với nhu cầu “tìm phương tiện di chuyển”, người dùng thường không chắc nên đi máy bay, tàu, xe hay thuê xe. Chatbot tách silo làm mất cơ hội so sánh đa phương án. |
| Trong flow thuê xe, khi người dùng hỏi “tôi muốn đi Hà Nội Hải Phòng ngày 3/6, có option nào rẻ và đánh giá cao”, chatbot chỉ đưa hướng dẫn đặt xe kèm ảnh minh hoạ, không trả về option cụ thể, giá, rating hay câu hỏi làm rõ. | <a href="Image/No_Unififed_Chatbot.png"><img src="Image/No_Unififed_Chatbot.png" alt="No_Unififed_Chatbot" width="160"></a> | `Low-confidence / Failure` | Chatbot nhận ra domain thuê xe nhưng không đáp ứng bài toán lựa chọn. Với người dùng, câu trả lời này không giúp ra quyết định. Cần thiết kế lại response thành dạng ranked options hoặc fallback có ích hơn. |

### Key insight rút ra từ self-use

Qua các thử nghiệm, Trip.com chatbot hiện tại hoạt động giống FAQ + redirect bot hơn là AI travel assistant. Nó có thể hướng người dùng đến đúng khu vực trong app/web, nhưng thường không:

- hiểu đầy đủ intent di chuyển
- hỏi thêm thông tin còn thiếu
- so sánh nhiều phương án
- gọi search để trả kết quả thật
- sửa câu trả lời khi người dùng phản hồi sai
- hoạt động thống nhất giữa `flight / train / car`

Điểm gãy quan trọng nhất cho build slice của nhóm là: người dùng muốn nói nhu cầu di chuyển bằng ngôn ngữ tự nhiên, nhưng chatbot hiện tại bắt người dùng tự chuyển nhu cầu đó thành thao tác tìm kiếm thủ công.
