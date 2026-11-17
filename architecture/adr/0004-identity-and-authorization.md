# ADR-0004: JWT và kiểm quyền trên từng tài nguyên

- Trạng thái: Đã áp dụng
- Ngày ghi nhận: 2026-11-17

## Bối cảnh

SportHub có Member, Receptionist, Coach, CenterManager và SystemAdministrator. Một role không đủ diễn đạt quyền: Coach chỉ được thao tác với lớp hoặc buổi mình được phân công; Member chỉ xem dữ liệu thuộc mình. Trang web có thể ẩn thao tác nhưng request vẫn có thể được gửi trực tiếp tới API.

## Quyết định và lý do

Backend xác thực request bằng **JWT bearer** và áp dụng policy theo role ở endpoint. Dịch vụ nghiệp vụ kiểm thêm chủ sở hữu, trạng thái tài khoản và quan hệ với tài nguyên theo từng request. Frontend dùng role để điều hướng và trình bày, còn API quyết định quyền thực tế. Đăng nhập email/mật khẩu và Google cùng đi qua Identity; mật khẩu được băm bằng BCrypt.

## Các lựa chọn đã cân nhắc

- **Chỉ kiểm role:** không ngăn Coach đọc lớp khác hoặc Member đọc hóa đơn người khác.
- **Chỉ bảo vệ route frontend:** không bảo vệ API khi client gửi request trực tiếp.
- **Session tập trung thay JWT:** có thể thu hồi tập trung, nhưng cần hạ tầng lưu và tra cứu session cho mỗi request. Thiết kế hiện tại đã chọn token và các kiểm tra trạng thái trong API.

## Hệ quả

Mỗi endpoint đọc/ghi dữ liệu riêng tư phải có cả policy phù hợp và kiểm phạm vi tài nguyên; không tin `memberId`/`coachId` do client tự gửi. Quyền có thể thay đổi sau khi token phát hành, nên những luồng nhạy cảm cần kiểm trạng thái hiện hành và kiểm thử trường hợp quyền bị thu hồi. Việc chỉ thấy một trang trong UI không chứng minh có quyền API.

## Nguồn kiểm chứng

- [JwtBearerExtensions.cs](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.BuildingBlocks/Infrastructure/Authentication/JwtBearerExtensions.cs), [Identity project](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/tree/main/backend/SportHub.Identity).
- [CoachDataScope.cs](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.Scheduling/Application/Services/CoachDataScope.cs), [MemberClassAccess.cs](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.Scheduling/Application/Services/MemberClassAccess.cs), [thiết kế quyền dữ liệu lớp](../Center-Management-System-Design-v3.md#53-khóa-cầu-lông-và-bóng-rổ).
