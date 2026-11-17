# UC-F02 — Lớp học, giữ chỗ và lịch sân

> Căn cứ: *SportHub SRS Part I* §2.2, §4.2 UC-16–21 và UC-28–46; UC-76 là gợi ý lịch phụ trợ. Khi khác nhau, áp dụng [Source of Truth](../requirements/00-Source-of-Truth.md), [Business Rules](../requirements/SportManagement_BusinessRules_v2.0_updated.docx) và [thiết kế hiện hành](../architecture/Center-Management-System-Design-v3.md). Flow 2 là phạm vi bắt buộc.

## Mục tiêu và actor

CenterManager cấu hình môn, phòng, Coach và lớp; Member/Receptionist giữ chỗ để ghi danh hoặc thuê sân; Coach báo vắng; hệ thống chống trùng lịch, đánh giá ngưỡng mở lớp và xử lý sự cố. Thanh toán/hoàn tiền thuộc [Flow 3](flow-03-payment-wallet-report.md); nội dung dạy học/điểm danh lớp thuộc [Flow 4](flow-04-training-attendance.md).

| UC SRS | Phạm vi | Kết quả |
| --- | --- | --- |
| UC-16–21 | Môn, phòng, giờ mở, Allow rental, chuyên môn, settings | Tài nguyên/cấu hình hợp lệ và có audit |
| UC-28–34 | Tạo/xuất bản lớp, Coach, buổi, báo vắng, xem lịch | Lịch lớp không trùng phòng/Coach |
| UC-35–39 | Ghi danh, hủy, ngưỡng, chuyển lớp, lịch cá nhân | Chỗ giữ có hạn; chỉ xác nhận sau thanh toán |
| UC-40–43 | Slot sân, thuê/hủy, check-in rental | Slot được giữ/cấp đúng quyền |
| UC-44–46 | Sự cố, tặng điểm bồi thường, thông báo | Tác động có lịch sử, xử lý và audit |

## Tiền điều kiện và bất biến

- Phòng vật lý không gắn cố định với môn; lớp chọn phòng Active. Phòng mới mặc định không cho thuê; `Allow rental` cần giờ và giá hợp lệ. Tắt cho thuê không xóa booking/lớp cũ. Coach được giao phải Active, đúng chuyên môn, không trùng lịch.
- Mọi ghi lịch phải qua occupancy backend trong transaction; khoảng dùng quy ước `[start, end)`. `ConfirmedCount <= ReservedCount <= Capacity`; `ReservedCount` bao gồm chỗ đang giữ.
- Enrollment là quyền cho cả khóa. Lớp `Published` hoặc `InProgress` còn buổi tương lai, còn chỗ và chưa đóng đăng ký nhận Member; giá trọn khóa, người vào giữa chừng học từ buổi kế tiếp.
- Hold checkout mặc định 15 phút, cấu hình 10–30 phút; hạn thực tế có thể sớm hơn khi sát giờ bắt đầu/hạn phản hồi. UI dùng `expiresAtUtc` server trả về.

## Luồng chính

### F02-A — Chuẩn bị lớp (UC-16–21, UC-28–34)

1. Manager tạo/sửa môn, phòng, giờ mở và block; quản lý chuyên môn Coach, thông số vận hành. Để cho thuê, Manager bật `Allow rental`, cấu hình giờ và giá theo phòng/khung giờ. Member chỉ thấy phòng đã bật và slot hợp lệ.
2. Manager tạo lớp Draft gồm môn, phòng Active, Coach phù hợp, lịch tuần, số buổi, sức chứa, học phí và chi phí. Hệ thống sinh `ClassSession`, kiểm giờ mở, block, xung đột phòng/Coach từng buổi. Coach không tự tạo/sửa lớp.
3. Hệ thống tính `BreakEvenThreshold = ceil(cost / price)`; chỉ Published khi ngưỡng không vượt Capacity và dữ liệu hợp lệ. Guest/Member xem lớp Published với lịch, giá, chỗ còn. Manager quản lý Draft, Published, InProgress, Completed/Cancelled theo điều kiện nghiệp vụ.

### F02-B — Ghi danh và lịch cá nhân (UC-35–36, UC-39)

1. Member hoặc Receptionist đã chọn Member yêu cầu ghi danh. Backend kiểm tài khoản Active, lớp mở, buổi tương lai, lịch lớp khác của Member và chỗ còn; tạo SeatHold và checkout trong transaction. Request đồng thời không được vượt Capacity.
2. Flow 3 thu điểm/VNPay. Sau thanh toán xác minh và kiểm lại hold, Enrollment thành Confirmed; Member thấy lịch của mình. Mua khi lớp InProgress vẫn thu giá cả khóa; buổi đã qua không tạo vắng mặt.
3. Member/nhân viên được phép tự hủy sau cảnh báo **không hoàn**. Trung tâm hủy thì xử lý theo Flow 3.

### F02-C — Ngưỡng và thay đổi lịch (UC-31–32, UC-37–38)

1. Ít nhất 72 giờ trước buổi đầu, job so Confirmed với ngưỡng. Nếu thiếu, đánh dấu AtRisk, báo Member/Manager/Coach. Member có tối đa 48 giờ, kết thúc ít nhất 24 giờ trước buổi đầu, để giữ chỗ, chuyển lớp cùng môn hoặc nhận hoàn điểm.
2. Ngưỡng đạt thì lớp được xác nhận và giữ người còn ghi danh, kể cả chưa phản hồi. Hết hạn vẫn thiếu và Manager chưa miễn ngưỡng có lý do/audit thì hủy, hoàn 100% điểm cho người còn ghi danh. Lựa chọn đã xử lý không bị đảo ngược; đánh giá muộn phải chốt hoặc dời khai giảng.
3. Coach chỉ báo vắng có lý do cho buổi được giao và rút khi Pending. Manager liên hệ Coach thay thế rồi quyết định dạy thay, dời hoặc hủy kèm buổi bù. Backend kiểm lại quyền, chuyên môn, phòng/Coach và lịch trống khi lưu; buổi không diễn ra được bù nguyên thời lượng và thông báo lịch cũ/mới.

### F02-D — Thuê sân và sự cố (UC-40–46)

1. Guest/Member xem slot của phòng cho thuê. Member hoặc Receptionist chọn Member, phòng, ngày và các block liên tiếp; backend kiểm hạn đặt trước, độ dài slot, tối đa 4 giờ, giờ mở, lớp/rental/block và giá theo phòng. Seed hiện hành: slot 60 phút, hôm nay + 3 ngày; giá trị thực lấy từ System Settings.
2. Checkout giữ slot; thanh toán xác minh mới xác nhận rental. Receptionist đánh dấu Checked-in/No-show; chưa đánh dấu có thể tự chuyển No-show sau giờ kết thúc. Sửa trạng thái đã ghi cần lý do/audit. Member tự hủy không hoàn.
3. Manager ghi sự cố với phạm vi/thời gian/lý do, xem trước người và dịch vụ bị ảnh hưởng, tạo block/fence rồi xử lý lớp/rental. Manager tự nhập buổi bù, backend kiểm xung đột khi lưu. Rental do trung tâm hủy có lựa chọn hoàn 100% qua Flow 3. Gym đóng tạm thời không kéo dài Membership; Manager chọn người bị ảnh hưởng, review số điểm/lý do rồi tặng điểm có chống trùng và audit. Thông báo đúng người nhận.

## Nhánh thay thế và lỗi

| Mã | Điều kiện | Trạng thái cuối |
| --- | --- | --- |
| A1 | Phòng/Coach không hợp lệ, xung đột, ngưỡng > Capacity | Không xuất bản/lưu; trả xung đột để Manager sửa. |
| A2 | Hai request lấy chỗ/slot cuối | Chỉ một giao dịch giữ được; request kia nhận conflict. |
| A3 | Hold hết hạn hoặc thanh toán đến muộn | Flow 3 QueryDR trước khi nhả; chỉ cấp nếu tài nguyên còn hợp lệ, nếu không bồi hoàn/hỗ trợ. |
| A4 | Chuyển lớp có chênh giá | Giữ chỗ đích, thanh toán/hoàn điểm phần chênh; lỗi thì giữ enrollment nguồn. |
| A5 | Lớp thiếu ngưỡng, Member im lặng | Không tự hoàn nếu lớp sau đó đạt ngưỡng; nếu hủy thì hoàn điểm, gửi thông báo và gợi ý lớp tương đương dạng snapshot, không tự giữ chỗ mới. |
| A6 | Coach tự đổi lịch; Manager lưu khi lịch đã thay | Từ chối quyền/xung đột; backend kiểm lại trong transaction. |
| A7 | Sự cố xử lý nhiều bước, rental đã trả tiền | Giữ lịch sử và fence có hạn; preview lại trước resolve. Toàn chuỗi UI không phải một transaction. |

## Hậu điều kiện và nghiệm thu

- Lớp, buổi, hold, Enrollment, rental và occupancy nhất quán; không quá sức chứa hoặc trùng phòng/Coach; thông báo/audit phản ánh quyết định có hiệu lực.
- Kiểm thử: tranh chỗ cuối, hết hạn hold, ghi danh lớp InProgress, ngưỡng đạt sau cảnh báo khi Member chưa trả lời, chuyển lớp có chênh, Coach báo vắng, tắt Allow rental với booking cũ, sự cố cắt qua lớp/rental/Gym, settings thay đổi giữa lúc xem và lưu.

## Truy vết

SRS §2.2/§4.2 UC-16–21, UC-28–46, UC-76; BR-20–25, BR-36–54, BR-60–70, BR-90, BR-94, BR-103–104, BR-116–118. Chi tiết route/DTO xem [API contract](../api/api-contract.md). UC-76 (AI gợi ý lịch Manager) chỉ là bản nháp, không tự lưu; tình trạng triển khai cần kiểm riêng và không thuộc chatbot Member/Coach của Flow 6.

### Kiểm kê UC trong SRS

| Mã | Tên UC trong SRS | Actor chính |
| --- | --- | --- |
| UC-16 | Manage sports | CenterManager |
| UC-17 | Manage rooms | CenterManager |
| UC-18 | Configure room availability | CenterManager |
| UC-19 | Enable court rental | CenterManager |
| UC-20 | Manage Coach specialties | CenterManager |
| UC-21 | Configure operating settings | CenterManager |
| UC-28 | Create class | CenterManager |
| UC-29 | Manage class status | CenterManager |
| UC-30 | Assign Coach | CenterManager |
| UC-31 | Manage class session | CenterManager |
| UC-32 | Handle Coach absence | Coach/CenterManager |
| UC-33 | View classes | Guest/Member |
| UC-34 | View Court Schedule | CenterManager/Receptionist |
| UC-35 | Enroll in class | Member/Receptionist |
| UC-36 | Cancel class enrollment | Member/Receptionist/CenterManager |
| UC-37 | Respond to class warning | Member |
| UC-38 | Evaluate class threshold | System/CenterManager |
| UC-39 | View personal schedule | Member |
| UC-40 | View free court slots | Guest/Member |
| UC-41 | Rent a court | Member/Receptionist |
| UC-42 | Cancel court rental | Member/Receptionist |
| UC-43 | Mark rental attendance | Receptionist |
| UC-44 | Create Incident Notice | CenterManager |
| UC-45 | Grant compensation points | CenterManager |
| UC-46 | Send manual notice | CenterManager |
| UC-76 | Get AI schedule suggestion | CenterManager; nhánh phụ trợ, kiểm tình trạng riêng |
