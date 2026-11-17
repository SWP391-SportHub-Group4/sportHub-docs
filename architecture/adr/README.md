# Architecture Decision Records (ADR)

ADR ghi lại **vì sao** SportHub chọn một hướng kiến trúc hoặc công nghệ, các lựa chọn đã cân nhắc và hệ quả của quyết định. [Thiết kế hệ thống](../Center-Management-System-Design-v3.md) mô tả trạng thái hiện tại và luồng chi tiết; ADR không thay thế tài liệu đó.

Các bản dưới đây được ghi nhận lại ngày **17/11/2026** từ mã nguồn và tài liệu hiện có. Đây là ngày lập hồ sơ, không mặc nhiên là ngày quyết định ban đầu. Liên kết mã nguồn trỏ về repo ứng dụng; khi triển khai thay đổi mới, cần đối chiếu với phiên bản code tương ứng.

| Mã | Quyết định | Trạng thái |
| --- | --- | --- |
| [ADR-0001](0001-modular-monolith.md) | Backend modular monolith | Đã áp dụng |
| [ADR-0002](0002-web-api-technology-stack.md) | Next.js, ASP.NET Core và hợp đồng HTTP | Đã áp dụng |
| [ADR-0003](0003-postgresql-ef-core.md) | PostgreSQL và EF Core cho dữ liệu nghiệp vụ | Đã áp dụng |
| [ADR-0004](0004-identity-and-authorization.md) | JWT và kiểm quyền trên từng tài nguyên | Đã áp dụng |
| [ADR-0005](0005-transactional-checkout.md) | Giữ chỗ, checkout và đối soát thanh toán | Đã áp dụng |
| [ADR-0006](0006-ai-service-boundary.md) | Tách tác vụ AI sang FastAPI, giữ quyền ở backend | Đã áp dụng |
| [ADR-0007](0007-in-process-background-jobs.md) | Tác vụ nền trong API và trạng thái bền vững | Đã áp dụng |

## Quy ước cập nhật

- Một ADR có mã tăng dần, tiêu đề, trạng thái, bối cảnh, quyết định, lựa chọn cân nhắc, hệ quả và nguồn kiểm chứng.
- ADR đã áp dụng được giữ lại để bảo toàn lịch sử. Nếu đổi hướng, tạo ADR mới và ghi rõ bản nào bị thay thế; cập nhật trạng thái và liên kết hai chiều.
- Phân biệt **quyết định đã triển khai** với đề xuất hoặc mục tiêu tương lai. Không suy ra đặc tính vận hành chưa được xác minh từ việc có thư viện hay cấu hình.

Mẫu ngắn:

```markdown
# ADR-XXXX: Tên quyết định

- Trạng thái: Đề xuất | Đã áp dụng | Bị thay thế
- Ngày ghi nhận: YYYY-MM-DD

## Bối cảnh
## Quyết định và lý do
## Các lựa chọn đã cân nhắc
## Hệ quả
## Nguồn kiểm chứng
```
