# UC-F01 — Tài khoản và Membership Gym

> Căn cứ: *SportHub SRS Part I* §2.1, §4.2 UC-01–15 và UC-22–27; đối chiếu [Source of Truth](../requirements/00-Source-of-Truth.md), [Business Rules](../requirements/SportManagement_BusinessRules_v2.0_updated.docx) và [API contract](../api/api-contract.md). Flow 1 là phạm vi bắt buộc của SRS.

## Mục tiêu và actor

Guest/Member tạo và dùng tài khoản; SystemAdministrator quản lý staff, role và trạng thái; CenterManager quản lý catalog Membership; Receptionist đăng ký Member tại quầy và ghi lượt vào/ra Gym. Thanh toán của gói và vé thuộc [Flow 3](flow-03-payment-wallet-report.md). Guest chỉ đọc nội dung công khai.

| UC SRS | Phạm vi | Kết quả |
| --- | --- | --- |
| UC-01–07 | Đăng ký, đăng nhập email/Google, mật khẩu, hồ sơ, nội dung công khai | Danh tính và dữ liệu cá nhân đúng quyền |
| UC-08–15 | Tạo/kích hoạt staff, trạng thái, tìm Member, audit | Tài khoản/role hợp lệ, thay đổi có audit |
| UC-22–27 | Catalog, mua/gia hạn, vé ngày, kiểm quyền, check-in | Quyền Gym đúng gói và thời hạn |

Ngoài phạm vi: PT 1:1, đổi Member thành staff hoặc ngược lại, gia hạn gói do Gym đóng tạm thời, hoàn tiền khi Member tự hủy.

## Tiền điều kiện và bất biến

- Email duy nhất. Member tự đăng ký cần OTP email; Google chỉ liên kết theo email đã xác minh. Chỉ Admin tạo staff; Coach cần chuyên môn. Receptionist/Manager chỉ tạo Member tại quầy.
- Role gồm `Member`, `Receptionist`, `Coach`, `CenterManager`, `SystemAdministrator`; Guest không phải role. Trạng thái tài khoản chỉ `ACTIVE`/`DEACTIVATED`, chờ kích hoạt là metadata riêng. Deactivate chặn request cần xác thực kế tiếp nhưng không tự xóa lịch, ví hay quyền lợi đã mua.
- Membership Gym độc lập với lớp/thuê sân. Gói bán có kỳ hạn 1/3/6/12 tháng; tên duy nhất, giá dương chia hết cho 1.000 VND. Giá/thời hạn đã bán được snapshot.
- Vé ngày cố định 50.000 VND, bán từ 06:00 đến trước 22:00 giờ Việt Nam khi Gym mở, hết hạn 22:00 cùng ngày và không hoàn. Từ 21:00 phải cảnh báo giờ đóng cửa.

## Luồng chính

### F01-A — Đăng ký, kích hoạt và đăng nhập (UC-01–05, UC-08–10, UC-13)

1. Guest nhập email, mật khẩu mạnh để đăng ký Member. Hệ thống gửi OTP; chỉ tạo tài khoản sau khi xác minh mã còn hạn. Đăng nhập Google liên kết tài khoản Active có email đã xác minh trùng, hoặc tạo Member mới chưa có mật khẩu.
2. Receptionist chọn đăng ký Member tại quầy, nhập tên, email, điện thoại. Hệ thống gửi link; Member tự kích hoạt và đặt mật khẩu, nhân viên không đặt hộ.
3. Admin tạo staff với role và chuyên môn nếu là Coach. Link kích hoạt hạn 24 giờ từ lúc tạo; gửi lại hoặc sửa email khi chưa kích hoạt làm vô hiệu link cũ nhưng không kéo dài hạn. Người nhận tự đặt mật khẩu hoặc liên kết Google.
4. Người dùng đăng nhập email/mật khẩu hoặc Google. Backend kiểm danh tính, trạng thái và role trước khi cấp JWT; frontend dẫn tới workspace phù hợp. Đổi/đặt lại mật khẩu đổi security stamp và vô hiệu phiên cũ theo quy tắc hiện hành.

### F01-B — Tài khoản và catalog (UC-06–07, UC-11–12, UC-14–15, UC-22–23)

1. Người dùng xem/sửa hồ sơ của mình; backend kiểm số điện thoại duy nhất và quyền ảnh. Receptionist/Manager tìm Member trong phạm vi phục vụ; Admin lọc tài khoản và xem chi tiết.
2. Admin Activate/Deactivate kèm lý do. Backend kiểm lại quyền và trạng thái trong giao dịch; chặn tự Deactivate và chặn vô hiệu Admin Active cuối. Đổi role chỉ giữa staff; không đổi role tài khoản đang là SystemAdministrator.
3. Manager tạo/sửa/ngừng bán gói. Guest/Member chỉ thấy gói đang bán; lịch sử và gói đã mua giữ nguyên. Audit log được giới hạn theo role.

### F01-C — Quyền Gym và check-in (UC-24–27)

1. Member hoặc Receptionist đã chọn đúng Member mua mới, gia hạn sớm hoặc chọn vé ngày rồi chuyển sang checkout Flow 3.
2. Chỉ khi backend xác minh thanh toán mới cấp quyền. Gói mới bắt đầu theo ngày thanh toán tại Việt Nam; gia hạn sớm bắt đầu ngày sau EndDate hiện có. EndDate theo tháng lịch và bao gồm ngày cuối. Thanh toán vé ngày sau 22:00 không tự cấp vé.
3. Member xem quyền lợi và hạn dùng. Receptionist tìm Member hoặc quét QR do server cấp, kiểm tài khoản Active, Gym mở và gói/vé còn hạn, ghi check-in rồi check-out. Job nhắc Member trước hạn Membership theo cấu hình.

## Nhánh thay thế và lỗi

| Mã | Điều kiện | Trạng thái cuối |
| --- | --- | --- |
| A1 | Email/phone trùng, OTP sai/hết hạn/quá số lần | Không tạo tài khoản; OTP đăng ký mặc định 10 phút, tối đa 5 lần sai, gửi lại sau 60 giây. |
| A2 | Quên mật khẩu | Luôn phản hồi trung tính; link reset dùng một lần, hạn 10 phút; không tiết lộ email có tài khoản. |
| A3 | Link kích hoạt sai/hết hạn | Không kích hoạt; chỉ người có quyền được gửi lại/sửa email. Member tại quầy chưa kích hoạt, không có dữ liệu sau 7 ngày được dọn và giải phóng email. |
| A4 | Deactivated hoặc đổi role bị cấm | Từ chối; giữ quyền lợi/đơn/lịch sử. Activate không tự phục hồi quyền lợi đã hủy. |
| A5 | Gói ngừng bán, Gym đóng, vé ngoài giờ, check-in không có quyền | Từ chối mua/check-in, không ghi lượt vào. |
| A6 | Member tự hủy Membership | Cảnh báo rõ không hoàn điểm/tiền; không tạo refund. Ngoại lệ lỗi trung tâm đi qua Flow 3. |

## Hậu điều kiện và nghiệm thu

- Tài khoản, role, trạng thái, link/OTP và audit khớp thao tác; API kiểm quyền kể cả khi frontend còn mở trang cũ.
- Chỉ thanh toán đã xác minh cấp gói/vé; giá và hạn đã mua giữ snapshot. Gia hạn không mất ngày còn lại. Check-in/out gắn đúng Member và nhân viên.
- Kiểm thử: OTP sai/hết hạn, gửi lại link không kéo hạn, Google email đã xác minh, Deactivate khi phiên còn mở, Admin cuối, gia hạn sát EndDate, vé lúc 21:00/22:00, Gym đóng, QR giả.

## Truy vết

SRS §2.1/§4.2 UC-01–15, 22–27; BR-1–19, BR-26–34, BR-44, BR-57, BR-71, BR-78, BR-88, BR-115–116. DTO và route xem [API contract](../api/api-contract.md); dữ liệu/giao dịch xem [thiết kế hệ thống](../architecture/Center-Management-System-Design-v3.md).

### Kiểm kê UC trong SRS

| Mã | Tên UC trong SRS | Actor chính |
| --- | --- | --- |
| UC-01 | Register Member account | Guest |
| UC-02 | Sign in with email | Người có tài khoản |
| UC-03 | Sign in with Google | Guest/Member |
| UC-04 | Reset forgotten password | Người có tài khoản |
| UC-05 | Manage password | Người đã đăng nhập |
| UC-06 | Manage profile | Người đã đăng nhập |
| UC-07 | Browse public content | Guest |
| UC-08 | Create staff account | SystemAdministrator |
| UC-09 | Manage staff account activation | SystemAdministrator |
| UC-10 | Activate staff account | Staff được mời |
| UC-11 | Manage account status | SystemAdministrator |
| UC-12 | View accounts | SystemAdministrator |
| UC-13 | Register Member at front desk | Receptionist |
| UC-14 | Search Member | Receptionist/CenterManager |
| UC-15 | View Audit Log | SystemAdministrator/CenterManager |
| UC-22 | View membership packages | Guest/Member |
| UC-23 | Manage membership packages | CenterManager |
| UC-24 | Purchase Gym membership | Member/Receptionist |
| UC-25 | Purchase Gym day pass | Member/Receptionist |
| UC-26 | Check membership status | Member/Receptionist |
| UC-27 | Record Gym attendance | Receptionist |
