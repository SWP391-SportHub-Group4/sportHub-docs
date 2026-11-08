# UC-F05 — AI workout recommendation

> Trạng thái: Draft for implementation · Owner: TODO · Cập nhật: TODO

## 1. Mục tiêu và phạm vi

- **Actor:** Member, Coach
- **Mục tiêu:** Member nhận gợi ý tập từ hồ sơ; Coach có thể xem, sửa và duyệt gợi ý trước khi lưu kế hoạch cho học viên được phân công.
- **Trigger:** Member yêu cầu gợi ý sau khi hoàn tất hồ sơ hoặc Coach yêu cầu gợi ý cho học viên hợp lệ.
- **Ngoài phạm vi:** TODO — ghi rõ chức năng liên quan nhưng không triển khai trong flow này.

## 2. Điều kiện

- **Tiền điều kiện:** TODO — phiên/role, cấu hình, dữ liệu và trạng thái đầu vào.
- **Hậu điều kiện thành công:** TODO — trạng thái nghiệp vụ, dữ liệu, audit và quyền lợi.
- **Hậu điều kiện thất bại:** TODO — rollback, giữ chỗ, thông báo hoặc retry.

## 3. Luồng chính

| Bước | Actor / hệ thống làm gì | Kết quả quan sát được | API / màn hình / bảng liên quan |
| --- | --- | --- | --- |
| 1 | Xác minh hồ sơ, môn, mục tiêu và quyền truy cập dữ liệu đầu vào. | Thiếu/không hợp lệ thì không gửi yêu cầu AI. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |
| 2 | Sinh gợi ý có thông số và giải thích; hiển thị nguồn gốc AI. | Người dùng phân biệt gợi ý AI với kế hoạch Coach. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |
| 3 | Member lưu theo cơ chế hiện hành; Coach rà soát/chỉnh sửa rồi mới lưu kế hoạch. | Không biến gợi ý chưa duyệt thành kế hoạch Coach. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |

## 4. Luồng thay thế và lỗi

| Mã | Điều kiện | Phản hồi và trạng thái cuối | Test |
| --- | --- | --- | --- |
| A1 | Hồ sơ hoặc goal thiếu, ngoài phạm vi hay phi thực tế. | TODO — status/error, rollback và thông báo | TODO |
| A2 | AI timeout/quota/provider lỗi hoặc trả nội dung không hợp lệ. | TODO — status/error, rollback và thông báo | TODO |
| A3 | Coach không có quyền với học viên hoặc bỏ qua bước review. | TODO — status/error, rollback và thông báo | TODO |

## 5. Hợp đồng triển khai

| Phần | Cần chốt từ code và yêu cầu |
| --- | --- |
| Frontend | Route, component/state, validation, loading/error, quyền hiển thị |
| API/backend | Method/path, request/response, error, authorization, idempotency |
| Data | Bảng và khóa liên quan, transaction, migration, audit/outbox |
| Tích hợp | Provider/callback, timeout/retry và dữ liệu nhạy cảm (nếu có) |

## 6. Tiêu chí nghiệm thu

- [ ] Không sinh gợi ý khi thiếu dữ liệu bắt buộc.
- [ ] Hiển thị giải thích và nhãn nguồn AI.
- [ ] Coach phải review trước khi lưu; semantics lưu của Member cần xác nhận rõ giữa trình duyệt và backend.

## 7. Kiểm thử triển khai

- [ ] Happy case được kiểm tra ở UI và API.
- [ ] Các nhánh lỗi/quyền quan trọng có test.
- [ ] Trạng thái sau thao tác khớp UI, API và DB; lưu bằng chứng/PR.

## 8. Truy vết và quyết định còn mở

- [Source of Truth](../requirements/00-Source-of-Truth.md) · [Business Rules](../requirements/SportManagement_BusinessRules_v2.0_updated.docx) · [API contract](../api/api-contract.md) · [Thiết kế hệ thống](../architecture/Center-Management-System-Design-v3.md) · [Demo 6 flow](../operations/LOCAL-DEMO-SCRIPT-6-FLOWS.md)
- **BR / endpoint / màn hình / test liên quan:** TODO.
- **Quyết định còn mở:** TODO.

> Dữ liệu fixture và đường bấm trong tài liệu demo là minh họa, không tự trở thành quy tắc nghiệp vụ.
