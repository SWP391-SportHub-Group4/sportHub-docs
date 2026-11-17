# ADR-0001: Backend modular monolith

- Trạng thái: Đã áp dụng
- Ngày ghi nhận: 2026-11-17

## Bối cảnh

SportHub phục vụ một trung tâm thể thao với các nghiệp vụ Identity, Membership, Scheduling, Training, Payment, Notification, AI, Audit và Administration. Đặt chỗ, hóa đơn, cấp quyền lợi và hoàn điểm thường cần thay đổi nhất quán trong cùng một thao tác. Nhóm cần phân chia code theo nghiệp vụ nhưng vẫn triển khai và vận hành với nguồn lực của đồ án.

## Quyết định và lý do

Dùng **một ASP.NET Core API** lắp ghép các project module. Mỗi module chứa controller, dịch vụ ứng dụng, quy tắc và cấu hình persistence thuộc phạm vi của mình; `SportHub.BuildingBlocks` chứa hợp đồng và thành phần chung. Các module phối hợp qua interface và cùng một `SportHubDbContext`/transaction khi nghiệp vụ cần tính nguyên tử. Cách này giữ ranh giới trong code, đồng thời cho phép giao dịch giữa checkout, giữ chỗ và fulfillment mà không cần giao thức phân tán.

## Các lựa chọn đã cân nhắc

- **Một project không chia module:** ít cấu trúc ban đầu nhưng dễ trộn quy tắc của các vai trò và luồng nghiệp vụ.
- **Microservices theo module:** cho phép triển khai độc lập nhưng thêm giao tiếp mạng, đồng bộ dữ liệu, quan sát và xử lý giao dịch phân tán. Phạm vi hiện tại chưa biện minh cho chi phí đó.

## Hệ quả

Triển khai backend như một đơn vị; thay đổi ở module này có thể ảnh hưởng module khác qua schema và interface chung. Ranh giới project không tự bảo đảm cô lập: cần review dependency, kiểm thử tích hợp và giữ các thao tác liên module trong service có chủ đích. Khi nhu cầu triển khai độc lập xuất hiện, có thể tách module sau khi xác định rõ quyền sở hữu dữ liệu và hợp đồng.

## Nguồn kiểm chứng

- [Thiết kế hệ thống, mục 3](../Center-Management-System-Design-v3.md#3-kiến-trúc-và-ranh-giới-module).
- [SportHub.API.csproj](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.API/SportHub.API.csproj) tham chiếu các module; [SportHubDbContext](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.API/Persistence/SportHubDbContext.cs) áp dụng cấu hình của từng module.
