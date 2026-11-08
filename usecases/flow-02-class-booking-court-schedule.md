# UC-F02 — Class booking & Court schedule

> Trạng thái: Draft for implementation · Owner: TODO · Cập nhật: TODO

## 1. Mục tiêu và phạm vi

- **Actor:** Guest, Center Manager, Member, Coach
- **Mục tiêu:** Member chọn lớp hoặc sân khả dụng; hệ thống giữ chỗ, xác nhận quyền sử dụng và hiển thị lịch đúng cho Member/Coach.
- **Trigger:** Member chọn lớp hoặc khung giờ thuê sân từ danh mục.
- **Ngoài phạm vi:** TODO — ghi rõ chức năng liên quan nhưng không triển khai trong flow này.

## 2. Điều kiện

- **Tiền điều kiện:** TODO — phiên/role, cấu hình, dữ liệu và trạng thái đầu vào.
- **Hậu điều kiện thành công:** TODO — trạng thái nghiệp vụ, dữ liệu, audit và quyền lợi.
- **Hậu điều kiện thất bại:** TODO — rollback, giữ chỗ, thông báo hoặc retry.

## 3. Luồng chính

| Bước | Actor / hệ thống làm gì | Kết quả quan sát được | API / màn hình / bảng liên quan |
| --- | --- | --- | --- |
| 1 | Guest/Member xem lớp, sân, giá và lịch còn chỗ. | Chỉ các lựa chọn khả dụng được trình bày. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |
| 2 | Member chọn lớp hoặc khung giờ; hệ thống kiểm sức chứa, lịch trùng, giá và giữ chỗ. | Quote/giữ chỗ hợp lệ hoặc lỗi có lý do. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |
| 3 | Sau checkout hợp lệ, hệ thống xác nhận ghi danh/thuê sân và cập nhật lịch. | Member thấy lịch; Coach thấy roster đúng quyền. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |

## 4. Luồng thay thế và lỗi

| Mã | Điều kiện | Phản hồi và trạng thái cuối | Test |
| --- | --- | --- | --- |
| A1 | Lớp hết chỗ hoặc sân đã bị giữ/đặt trước. | TODO — status/error, rollback và thông báo | TODO |
| A2 | Khung giờ ngoài giờ mở cửa hoặc trùng lịch nguồn lực. | TODO — status/error, rollback và thông báo | TODO |
| A3 | Giữ chỗ hết hạn hoặc thanh toán chưa xác nhận. | TODO — status/error, rollback và thông báo | TODO |

## 5. Hợp đồng triển khai

| Phần | Cần chốt từ code và yêu cầu |
| --- | --- |
| Frontend | Route, component/state, validation, loading/error, quyền hiển thị |
| API/backend | Method/path, request/response, error, authorization, idempotency |
| Data | Bảng và khóa liên quan, transaction, migration, audit/outbox |
| Tích hợp | Provider/callback, timeout/retry và dữ liệu nhạy cảm (nếu có) |

## 6. Tiêu chí nghiệm thu

- [ ] Không phát sinh hai booking xung đột trên cùng nguồn lực/khung giờ.
- [ ] Quyền sử dụng chỉ được cấp sau điều kiện checkout hợp lệ.
- [ ] Member và Coach nhìn thấy lịch/roster đúng phạm vi quyền.

## 7. Kiểm thử triển khai

- [ ] Happy case được kiểm tra ở UI và API.
- [ ] Các nhánh lỗi/quyền quan trọng có test.
- [ ] Trạng thái sau thao tác khớp UI, API và DB; lưu bằng chứng/PR.

## 8. Truy vết và quyết định còn mở

- [Source of Truth](../requirements/00-Source-of-Truth.md) · [Business Rules](../requirements/SportManagement_BusinessRules_v2.0_updated.docx) · [API contract](../api/api-contract.md) · [Thiết kế hệ thống](../architecture/Center-Management-System-Design-v3.md) · [Demo 6 flow](../operations/LOCAL-DEMO-SCRIPT-6-FLOWS.md)
- **BR / endpoint / màn hình / test liên quan:** TODO.
- **Quyết định còn mở:** TODO.

> Dữ liệu fixture và đường bấm trong tài liệu demo là minh họa, không tự trở thành quy tắc nghiệp vụ.
