# ADR-0005: Giữ chỗ và checkout có giao dịch, đối soát thanh toán

- Trạng thái: Đã áp dụng
- Ngày ghi nhận: 2026-11-17

## Bối cảnh

Một lượt mua lớp hoặc thuê sân phải giữ tài nguyên trong lúc khách thanh toán. Callback VNPay có thể lặp, đến muộn hoặc đến sau khi hold hết hạn. Việc thu tiền và cấp quyền lợi không diễn ra trong cùng giao dịch database với cổng thanh toán bên ngoài.

## Quyết định và lý do

Tạo **invoice/checkout và hold có thời hạn** trước khi thanh toán; lưu giá và thông tin quyền lợi tại thời điểm mua. Các thao tác nội bộ dùng transaction, khóa dòng/cập nhật có điều kiện và idempotency key. Chỉ cấp dịch vụ sau khi backend xác minh thanh toán và kiểm lại hold, sức chứa, lịch và hạn sử dụng. Tác vụ hết hạn và đối soát xử lý phiên chưa hoàn tất; tiền đã xác minh nhưng không thể cấp quyền lợi đi vào trạng thái cần bồi hoàn/xử lý thay vì âm thầm coi là đã hoàn thành.

## Các lựa chọn đã cân nhắc

- **Cấp chỗ ngay khi tạo URL thanh toán:** giữ chỗ vô hạn cho giao dịch bỏ dở.
- **Không giữ chỗ trước khi thanh toán:** có nguy cơ thu tiền xong nhưng hết chỗ; vẫn phải có nhánh bồi hoàn.
- **Tin callback hoặc trạng thái frontend:** dễ cấp trùng hoặc cấp dịch vụ khi chưa xác minh nguồn thanh toán.

## Hệ quả

Luồng thanh toán có nhiều trạng thái và job đối soát; UI phải hiển thị rõ đang chờ, đã cấp, đã bồi hoàn hoặc cần xử lý. Các nhánh retry/callback phải idempotent, có kiểm thử đồng thời và kịch bản thanh toán đến muộn. Không thể bảo đảm một giao dịch ACID bao trùm VNPay; hệ thống dùng xác minh, đối soát và bồi hoàn để xử lý phần ngoài database.

## Nguồn kiểm chứng

- [Thiết kế hệ thống, mục 6](../Center-Management-System-Design-v3.md#6-payment-vnpay-và-ví-điểm), [Use case payment](../../usecases/flow-03-payment-wallet-report.md).
- [CheckoutService.cs](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.Payment/Application/Services/CheckoutService.cs), [ClassEnrollmentFulfillment.cs](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/main/backend/SportHub.Scheduling/Application/Services/ClassEnrollmentFulfillment.cs).
