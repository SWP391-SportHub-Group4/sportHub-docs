# ADR-0007: Tác vụ nền trong API, trạng thái lưu ở database

- Trạng thái: Đã áp dụng
- Ngày ghi nhận: 2026-11-17

## Bối cảnh

Giữ chỗ, checkout, vé ngày, lời mời, thông báo và đối soát có hạn xử lý hoặc cần thử lại sau request. Việc buộc trình duyệt đang mở hoặc request kế tiếp kích hoạt các bước này sẽ khiến trạng thái chậm cập nhật.

## Quyết định và lý do

Đăng ký các **hosted service** trong ASP.NET Core API để quét và xử lý tác vụ định kỳ. Trạng thái nghiệp vụ, deadline và kết quả xử lý nằm trong PostgreSQL; job đọc lại trạng thái thay vì dựa vào bộ nhớ của tiến trình. Cách này tận dụng cùng service, DI và transaction hiện có, phù hợp quy mô triển khai hiện tại.

## Các lựa chọn đã cân nhắc

- **Chạy khi có request:** tiết kiệm worker nhưng không bảo đảm hạn xử lý khi ít người dùng.
- **Message broker/worker riêng:** mở rộng và cô lập tốt hơn nhưng thêm hạ tầng, vận hành và giao thức giao việc chưa cần cho phạm vi hiện tại.

## Hệ quả

API phải hoạt động để job chạy. Một lần quét có thể bị ngắt hoặc chạy lại; mỗi handler cần kiểm tra trạng thái và bảo đảm xử lý lặp an toàn. Khi tăng số bản sao API, phải xác nhận cơ chế tranh chấp/khóa của từng job trước khi bật nhiều worker. Không suy ra bảo đảm chạy đúng một lần chỉ từ việc đăng ký `AddHostedService`.

## Nguồn kiểm chứng

- [Program.cs](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.API/Program.cs) đăng ký `SeatHoldExpiryJob`, `CheckoutExpiryJob`, `PaymentReconciliationJob`, `NotificationDispatchJob` và các job khác.
- [Thiết kế hệ thống, mục 6.3](../Center-Management-System-Design-v3.md#63-expiry-và-đối-soát), [RUNBOOK](../../operations/RUNBOOK.md).
