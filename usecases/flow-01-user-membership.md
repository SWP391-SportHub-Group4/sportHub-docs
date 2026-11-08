# UC-F01 — User & Membership

> Trạng thái: Draft for implementation · Owner: TODO · Cập nhật: TODO

## 1. Mục tiêu và phạm vi

- **Actor:** Guest, System Administrator, Center Manager, Member, Receptionist
- **Mục tiêu:** Người dùng được xác thực/phân quyền đúng; Member có quyền lợi gói Gym hợp lệ và Receptionist kiểm tra được khi check-in.
- **Trigger:** Guest tìm dịch vụ hoặc một vai trò bắt đầu tác vụ tài khoản, gói tập hay check-in.
- **Ngoài phạm vi:** TODO — ghi rõ chức năng liên quan nhưng không triển khai trong flow này.

## 2. Điều kiện

- **Tiền điều kiện:** TODO — phiên/role, cấu hình, dữ liệu và trạng thái đầu vào.
- **Hậu điều kiện thành công:** TODO — trạng thái nghiệp vụ, dữ liệu, audit và quyền lợi.
- **Hậu điều kiện thất bại:** TODO — rollback, giữ chỗ, thông báo hoặc retry.

## 3. Luồng chính

| Bước | Actor / hệ thống làm gì | Kết quả quan sát được | API / màn hình / bảng liên quan |
| --- | --- | --- | --- |
| 1 | Guest xem danh mục công khai; chọn đăng ký hoặc đăng nhập. | Danh mục công khai; màn xác thực. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |
| 2 | Hệ thống xác thực tài khoản và xác định vai trò; Admin/Manager thao tác trong phạm vi quyền. | Phiên hợp lệ; thao tác bị chặn nếu sai quyền. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |
| 3 | Member xem gói Gym và quyền lợi; Receptionist kiểm tra điều kiện rồi xác nhận check-in. | Quyền lợi và lượt check-in được phản ánh đúng. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |

## 4. Luồng thay thế và lỗi

| Mã | Điều kiện | Phản hồi và trạng thái cuối | Test |
| --- | --- | --- | --- |
| A1 | Email/Google sign-in hoặc khôi phục mật khẩu thất bại. | TODO — status/error, rollback và thông báo | TODO |
| A2 | Tài khoản Deactivated, sai vai trò hoặc Admin tự vô hiệu hóa. | TODO — status/error, rollback và thông báo | TODO |
| A3 | Gói hết hạn, chưa thanh toán hoặc check-in lặp. | TODO — status/error, rollback và thông báo | TODO |

## 5. Hợp đồng triển khai

| Phần | Cần chốt từ code và yêu cầu |
| --- | --- |
| Frontend | Route, component/state, validation, loading/error, quyền hiển thị |
| API/backend | Method/path, request/response, error, authorization, idempotency |
| Data | Bảng và khóa liên quan, transaction, migration, audit/outbox |
| Tích hợp | Provider/callback, timeout/retry và dữ liệu nhạy cảm (nếu có) |

## 6. Tiêu chí nghiệm thu

- [ ] Guest chỉ thấy nội dung công khai; dữ liệu cá nhân yêu cầu đăng nhập đúng vai trò.
- [ ] Quyền lợi Gym chỉ được dùng khi trạng thái và thời hạn hợp lệ.
- [ ] Check-in thành công được ghi đúng một lần; trường hợp không hợp lệ trả lý do rõ ràng.

## 7. Kiểm thử triển khai

- [ ] Happy case được kiểm tra ở UI và API.
- [ ] Các nhánh lỗi/quyền quan trọng có test.
- [ ] Trạng thái sau thao tác khớp UI, API và DB; lưu bằng chứng/PR.

## 8. Truy vết và quyết định còn mở

- [Source of Truth](../requirements/00-Source-of-Truth.md) · [Business Rules](../requirements/SportManagement_BusinessRules_v2.0_updated.docx) · [API contract](../api/api-contract.md) · [Thiết kế hệ thống](../architecture/Center-Management-System-Design-v3.md) · [Demo 6 flow](../operations/LOCAL-DEMO-SCRIPT-6-FLOWS.md)
- **BR / endpoint / màn hình / test liên quan:** TODO.
- **Quyết định còn mở:** TODO.

> Dữ liệu fixture và đường bấm trong tài liệu demo là minh họa, không tự trở thành quy tắc nghiệp vụ.
