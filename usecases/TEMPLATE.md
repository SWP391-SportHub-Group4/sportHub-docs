# UC-FXX — Tên flow

> Trạng thái: Draft · Owner: TODO · Cập nhật: TODO

## 1. Mục tiêu và phạm vi

- **Actor:** TODO
- **Mục tiêu:** TODO
- **Trigger:** TODO
- **Trong phạm vi / ngoài phạm vi:** TODO

## 2. Điều kiện

- **Tiền điều kiện:** TODO (tài khoản, quyền, dữ liệu, cấu hình).
- **Hậu điều kiện thành công:** TODO (trạng thái, dữ liệu, quyền lợi, audit).
- **Hậu điều kiện thất bại:** TODO (rollback, giữ chỗ, thông báo).

## 3. Luồng chính

| Bước | Actor / hệ thống làm gì | Kết quả quan sát được | API / màn hình / bảng liên quan |
| --- | --- | --- | --- |
| 1 | TODO | TODO | TODO |

## 4. Luồng thay thế và lỗi

| Mã | Điều kiện | Phản hồi và trạng thái cuối | Test |
| --- | --- | --- | --- |
| A1 | TODO | TODO | TODO |

## 5. Hợp đồng triển khai

| Phần | Cần chốt |
| --- | --- |
| Frontend | Route, state, validation, loading/error, quyền hiển thị |
| API/backend | Method/path, request, response, lỗi, phân quyền, idempotency |
| Data | Bảng, transaction, migration, audit/outbox |
| Tích hợp | Provider, callback, timeout/retry, dữ liệu nhạy cảm |

## 6. Tiêu chí nghiệm thu và kiểm thử

- [ ] Happy case có bằng chứng FE và API.
- [ ] Các nhánh lỗi/quyền quan trọng có test.
- [ ] Dữ liệu sau thao tác khớp UI, API và DB.

## 7. Truy vết và việc cần chốt

- [Source of Truth](../requirements/00-Source-of-Truth.md) · [Business Rules](../requirements/SportManagement_BusinessRules_v2.0_updated.docx) · [API contract](../api/api-contract.md) · [Thiết kế hệ thống](../architecture/Center-Management-System-Design-v3.md) · [Demo 6 flow](../operations/LOCAL-DEMO-SCRIPT-6-FLOWS.md)
- **BR / endpoint / màn hình / test liên quan:** TODO.
- **Quyết định còn mở:** TODO.

> Dữ liệu fixture và đường bấm trong tài liệu demo là minh họa, không tự trở thành quy tắc nghiệp vụ.
