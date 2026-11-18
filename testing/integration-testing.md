# Integration Testing

### 5.1 Ma trận tích hợp

Các file sau đã được tìm thấy; danh sách là điểm truy vết test code, không phải xác nhận kết quả chạy mới.

| ID | Thành phần và tình huống | Assertion trọng tâm | Test code tham chiếu |
| --- | --- | --- | --- |
| IT-01 | Identity → JWT → database auth state | Login đúng; token cũ mất hiệu lực khi tài khoản/quyền thay đổi | `backend/SportHub.Security.Tests/Integration/LoginContractTests.cs`, `AccountStatusJwtTests.cs` |
| IT-02 | Scheduling → hold → PostgreSQL transaction | Không bán vượt; không trùng tài nguyên/Member | `backend/SportHub.Scheduling.Tests/Integration/SeatHoldConcurrencyTests.cs`, `OccupancyConcurrencyTests.cs` |
| IT-03 | Checkout → payment → wallet → membership/enrollment/rental | Amount đúng; cấp một lần; dữ liệu rollback nhất quán | `backend/SportHub.Payment.Tests/Integration/FullPaymentAndActivationTests.cs`, `CheckoutFlowTests.cs`, `WalletConcurrencyTests.cs` |
| IT-04 | Gateway event → verification → reconciliation | Chống giả mạo/replay; xử lý thanh toán muộn | `backend/SportHub.Payment.Tests/Integration/PaymentReturnVerificationTests.cs`, `FrontDeskPaymentReconciliationTests.cs` |
| IT-05 | Incident → schedule → refund/notice | Fence, cập nhật lịch và quyền lợi nhất quán | `backend/SportHub.Scheduling.Tests/Integration/IncidentSessionRecoveryTests.cs`, `backend/SportHub.Payment.Tests/Integration/IncidentRefundTests.cs` |
| IT-06 | Training → enrollment → authorization | Kết thúc/duy trì quan hệ đúng; quyền lịch sử và phạm vi Coach | `backend/SportHub.Training.Tests/Integration/RelationshipLifecycleTests.cs`, `backend/SportHub.Security.Tests/Integration/CoachMemberReadScopeTests.cs` |
| IT-07 | Business transaction → outbox → dispatcher | Tạo event cùng commit; retry/dedup | `backend/SportHub.Payment.Tests/Integration/OutboxDispatchTests.cs` |
| IT-08 | Ledger/invoice → report/export | Kỳ báo cáo, snapshot và đối soát thống nhất | `backend/SportHub.Payment.Tests/Integration/ReportReconciliationTests.cs`, `ReportExportParityTests.cs` |
| IT-09 | AI member/coach routes → fake model | Tách wrapper theo vai trò; route không tồn tại bị từ chối | `ai-service/tests/test_role_routes.py` |

### 5.2 Quy trình và giới hạn

1. Seed dữ liệu với ID riêng; ghi số dư, số chỗ, trạng thái và số bản ghi trước thao tác.
2. Thực hiện qua HTTP hoặc service đúng cấp độ mà test đang kiểm tra; không gọi service rồi mô tả như đã đi qua toàn bộ HTTP middleware.
3. Với concurrency, gửi các thao tác đồng thời, kiểm tra cả response và trạng thái cuối trong database.
4. Với retry, gửi lại request/callback và kiểm tra Payment, ledger, quyền lợi, audit/outbox không phát sinh sai.
5. Lưu result và tên test cụ thể; teardown theo vòng đời fixture, không xóa database dùng chung.

Chạy integration với PostgreSQL thật chứng minh hành vi transaction trong cấu hình test; không tự chứng minh mạng public, provider thật, triển khai production hoặc E2E toàn hệ thống.

[Mục lục tài liệu kiểm thử](README.md)
