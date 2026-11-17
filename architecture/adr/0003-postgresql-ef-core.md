# ADR-0003: PostgreSQL và EF Core cho dữ liệu nghiệp vụ

- Trạng thái: Đã áp dụng
- Ngày ghi nhận: 2026-11-17

## Bối cảnh

Tài khoản, lịch sân/lớp, giữ chỗ, hóa đơn và ví điểm có quan hệ chặt và các bất biến cần bảo vệ khi nhiều request đến đồng thời. Các báo cáo cũng cần truy vấn quan hệ. Một hệ lưu trữ có giao dịch và ràng buộc ở database phù hợp hơn việc chỉ kiểm tra ở giao diện.

## Quyết định và lý do

Dùng **PostgreSQL 16** làm cơ sở dữ liệu vận hành và **EF Core 10 với Npgsql** để ánh xạ entity, cấu hình schema và migration. API dùng một `SportHubDbContext` có cấu hình từ các module. Các đường ghi nhạy cảm sử dụng transaction và khi cần dùng SQL PostgreSQL như `FOR UPDATE` hoặc cập nhật có điều kiện để giữ bất biến khi cạnh tranh. Schema/migration được quản lý cùng mã nguồn.

## Các lựa chọn đã cân nhắc

- **Lưu trữ không quan hệ:** linh hoạt schema nhưng khó diễn đạt quan hệ và bảo đảm đồng thời cho lịch, hóa đơn, ví.
- **Chỉ dùng ORM và kiểm tra ở ứng dụng:** đơn giản ở đoạn code thường, nhưng kiểm tra rồi ghi riêng lẻ có thể gây overbooking hoặc ghi trùng khi nhiều request chạy cùng lúc.

## Hệ quả

Các module chia sẻ database nên migration phải được review như thay đổi hợp đồng; không được giả định mọi thay đổi schema độc lập. SQL đặc thù PostgreSQL tăng độ chính xác cho các luồng cạnh tranh nhưng giảm khả năng đổi database. Test tích hợp cần PostgreSQL thật, không chỉ database in-memory.

## Nguồn kiểm chứng

- [docker-compose.yml](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/docker-compose.yml), [Program.cs](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.API/Program.cs), [SportHubDbContext.cs](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.API/Persistence/SportHubDbContext.cs).
- [CheckoutService.cs](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.Payment/Application/Services/CheckoutService.cs), [ClassEnrollmentFulfillment.cs](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.Scheduling/Application/Services/ClassEnrollmentFulfillment.cs).
