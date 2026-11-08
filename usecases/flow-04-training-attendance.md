# UC-F04 — Training & attendance

> Trạng thái: Draft for implementation · Owner: TODO · Cập nhật: TODO

## 1. Mục tiêu và phạm vi

- **Actor:** Coach, Member
- **Mục tiêu:** Coach được phân công ghi kế hoạch, điểm danh và kết quả đúng buổi; Member xem dữ liệu của mình.
- **Trigger:** Coach mở lớp/buổi được phân công hoặc Member mở kế hoạch, kết quả.
- **Ngoài phạm vi:** TODO — ghi rõ chức năng liên quan nhưng không triển khai trong flow này.

## 2. Điều kiện

- **Tiền điều kiện:** TODO — phiên/role, cấu hình, dữ liệu và trạng thái đầu vào.
- **Hậu điều kiện thành công:** TODO — trạng thái nghiệp vụ, dữ liệu, audit và quyền lợi.
- **Hậu điều kiện thất bại:** TODO — rollback, giữ chỗ, thông báo hoặc retry.

## 3. Luồng chính

| Bước | Actor / hệ thống làm gì | Kết quả quan sát được | API / màn hình / bảng liên quan |
| --- | --- | --- | --- |
| 1 | Coach mở lớp và danh sách học viên thuộc phạm vi được giao. | Chỉ lớp/buổi và Member được phép xuất hiện. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |
| 2 | Coach tạo/cập nhật kế hoạch, điểm danh hoặc kết quả theo buổi hợp lệ. | Dữ liệu gắn đúng lớp, buổi và học viên. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |
| 3 | Member mở Training/Schedule để xem kế hoạch, điểm danh và kết quả của mình. | Dữ liệu được phản chiếu đúng quyền, không cho sửa nội dung Coach. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |

## 4. Luồng thay thế và lỗi

| Mã | Điều kiện | Phản hồi và trạng thái cuối | Test |
| --- | --- | --- | --- |
| A1 | Coach không còn được phân công hoặc chỉ là Coach dạy thay của buổi khác. | TODO — status/error, rollback và thông báo | TODO |
| A2 | Buổi chưa/mất cửa sổ điểm danh hoặc dữ liệu kết quả không hợp lệ. | TODO — status/error, rollback và thông báo | TODO |
| A3 | Member cố truy cập dữ liệu của người khác. | TODO — status/error, rollback và thông báo | TODO |

## 5. Hợp đồng triển khai

| Phần | Cần chốt từ code và yêu cầu |
| --- | --- |
| Frontend | Route, component/state, validation, loading/error, quyền hiển thị |
| API/backend | Method/path, request/response, error, authorization, idempotency |
| Data | Bảng và khóa liên quan, transaction, migration, audit/outbox |
| Tích hợp | Provider/callback, timeout/retry và dữ liệu nhạy cảm (nếu có) |

## 6. Tiêu chí nghiệm thu

- [ ] Quyền Coach được kiểm lại ở backend cho từng lớp/buổi.
- [ ] Kế hoạch, attendance và result tham chiếu đúng học viên/buổi.
- [ ] Member chỉ đọc dữ liệu cá nhân được cấp quyền.

## 7. Kiểm thử triển khai

- [ ] Happy case được kiểm tra ở UI và API.
- [ ] Các nhánh lỗi/quyền quan trọng có test.
- [ ] Trạng thái sau thao tác khớp UI, API và DB; lưu bằng chứng/PR.

## 8. Truy vết và quyết định còn mở

- [Source of Truth](../requirements/00-Source-of-Truth.md) · [Business Rules](../requirements/SportManagement_BusinessRules_v2.0_updated.docx) · [API contract](../api/api-contract.md) · [Thiết kế hệ thống](../architecture/Center-Management-System-Design-v3.md) · [Demo 6 flow](../operations/LOCAL-DEMO-SCRIPT-6-FLOWS.md)
- **BR / endpoint / màn hình / test liên quan:** TODO.
- **Quyết định còn mở:** TODO.

> Dữ liệu fixture và đường bấm trong tài liệu demo là minh họa, không tự trở thành quy tắc nghiệp vụ.
