# UC-F06 — Trợ lý AI theo ngữ cảnh người dùng

> Căn cứ: *SportHub SRS Part I* §2.6, §4.2 UC-75 (phạm vi mở rộng), đối chiếu [Source of Truth](../requirements/00-Source-of-Truth.md#ai-flow-56--cập-nhật-đối-chiếu-sau-các-bản-vá-02112026), [Functional & Nonfunctional Specification](../requirements/Functional-and-Nonfunctional-Spec.md#bổ-sung-yêu-cầu-ai-flow-56--02112026) và [bằng chứng AI](../verification/AI-Member-Coach-Implementation-Verification.md). UC-76 (Manager xin lịch AI) được truy vết ở [Flow 2](flow-02-class-booking-court-schedule.md), không phải chatbox này.

## Mục tiêu và actor

Member hoặc Coach hỏi bằng ngôn ngữ tự nhiên về trang/chức năng của mình, lịch, lớp và dịch vụ trung tâm. Backend tạo context **chỉ đọc** theo người đang đăng nhập; AI trả lời trong phạm vi đó. Chatbox UI chỉ dành cho Member và Coach. Manager, Receptionist, Admin không có chatbox theo Flow 6 hiện hành.

## Tiền điều kiện và ranh giới quyền

- Request có phiên hợp lệ; backend lấy user/role từ phiên, không tin role hoặc owner do prompt gửi lên. Member chỉ có ngữ cảnh lớp, Membership, hóa đơn, ví, lịch và tính năng của chính mình. Coach chỉ có lớp/buổi được giao và nội dung giảng dạy trong phạm vi hiện hành.
- Hướng dẫn navigation phải dựa trên page/tab/button/route đã xác minh từ frontend cho đúng role. Endpoint API không tự trở thành nút giao diện. Với đường đi chưa xác minh, trợ lý nói rõ giới hạn thay vì bịa bước bấm.
- Prompt đang published do `ISystemPromptReader` cung cấp; request tiếp nối phải giữ kiểm tra quyền và phiên bản prompt. Câu hỏi tiếng Việt/tiếng Anh đều hợp lệ; câu trả lời có thể dùng tiếng Việt nhưng giữ nguyên nhãn button tiếng Anh của UI.
- Chat không gọi chức năng ghi nghiệp vụ: không thanh toán, hoàn tiền, hủy, publish hay sửa dữ liệu. Không đưa lời khuyên y tế, pháp lý, tài chính cá nhân như kết luận chuyên môn.

## Luồng chính UC-75

1. Member bấm nút mascot ở workspace hoặc Coach mở chat trong workspace, nhập câu hỏi về navigation hay dữ liệu nghiệp vụ thuộc mình.
2. Backend xác thực phiên/role, phân loại câu hỏi và dựng context read-only theo dữ liệu được phép tại thời điểm hỏi; lấy navigation catalog đã xác minh và published system prompt. Không gửi dữ liệu của role/chủ thể khác cho AI service.
3. AI service trả câu trả lời dựa trên context và chủ đề được phép. Backend kiểm giới hạn đầu ra, ghi AI log theo chính sách, trả lời kèm các bước navigation có thật nếu đã xác minh. Câu hỏi lịch theo SRS có mục tiêu phản hồi trong 3 giây; cần đo trên môi trường nghiệm thu, không tự coi là SLA production đã đạt.
4. Người dùng xem câu trả lời và có thể mở trang nghiệp vụ để xác nhận. Muốn thay đổi dữ liệu, họ phải dùng form/API nghiệp vụ được phân quyền; nội dung chat không tự thực hiện lệnh.

## Nhánh thay thế và lỗi

| Mã | Điều kiện | Trạng thái cuối |
| --- | --- | --- |
| A1 | Phiên hết hạn, role ngoài Member/Coach, quyền lớp thay đổi | Từ chối request hoặc loại dữ liệu đã mất quyền; không dùng context cũ. |
| A2 | Hỏi dữ liệu người khác hoặc yêu cầu bỏ qua policy qua prompt | Không đưa dữ liệu đó vào context; từ chối phần vượt quyền. |
| A3 | Hỏi chức năng không có/không xác minh được trong frontend | Nói không có hướng dẫn đã kiểm chứng, không bịa tên menu/button. |
| A4 | Hỏi thanh toán/hoàn/hủy/sửa qua chat | Chỉ hướng dẫn mở màn hình được phép nếu có; không gọi mutation. |
| A5 | Hỏi tư vấn y tế/pháp lý/tài chính hoặc thông tin ngoài phạm vi | Trả giới hạn phù hợp; không phát biểu như chuyên gia. |
| A6 | Provider timeout/quota/lỗi hoặc câu trả lời không phù hợp | Trả lỗi có kiểm soát; không trình bày câu trả lời suy đoán như dữ liệu đã xác minh. |
| A7 | Prompt published đổi giữa hai lượt hỏi | Kiểm phiên bản/quyền khi tiếp nối; không tiếp tục bằng context hoặc policy lỗi thời. |

## Hậu điều kiện và nghiệm thu

- Chat chỉ đọc; không có thay đổi invoice, lịch, attendance hay hồ sơ do câu hỏi. Context/log nằm trong phạm vi quyền, navigation phản ánh UI thực tế.
- Kiểm thử: Member hỏi invoice/wallet và thử trang Coach; Coach hỏi lớp được giao và lớp khác; tiếng Việt có dấu/không dấu, tiếng Anh và hỗn hợp; nhãn button tiếng Anh; prompt injection; câu hỏi không có đường đi xác minh; provider lỗi; thay prompt version; đổi quyền giữa các lượt; đo thời gian câu hỏi lịch theo SRS.

## Truy vết

SRS §2.6/§4.2 UC-75; BR-105–109; FR-AI-03/04/05 và NFR-AI-01. Hợp đồng xem [API contract](../api/api-contract.md). UC-76 là chức năng Manager đề xuất lịch lớp, phải được Manager review và backend kiểm lại occupancy trước khi lưu; không cấp quyền này cho chatbox Member/Coach.
