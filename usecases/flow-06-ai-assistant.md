# UC-F06 — AI assistant

> Trạng thái: Draft for implementation · Owner: TODO · Cập nhật: TODO

## 1. Mục tiêu và phạm vi

- **Actor:** Member, Coach
- **Mục tiêu:** Assistant trả lời dựa trên ngữ cảnh người đang đăng nhập, không lộ dữ liệu khác vai trò và không tự thực hiện thao tác thay người dùng.
- **Trigger:** Member hoặc Coach mở chat và gửi câu hỏi.
- **Ngoài phạm vi:** TODO — ghi rõ chức năng liên quan nhưng không triển khai trong flow này.

## 2. Điều kiện

- **Tiền điều kiện:** TODO — phiên/role, cấu hình, dữ liệu và trạng thái đầu vào.
- **Hậu điều kiện thành công:** TODO — trạng thái nghiệp vụ, dữ liệu, audit và quyền lợi.
- **Hậu điều kiện thất bại:** TODO — rollback, giữ chỗ, thông báo hoặc retry.

## 3. Luồng chính

| Bước | Actor / hệ thống làm gì | Kết quả quan sát được | API / màn hình / bảng liên quan |
| --- | --- | --- | --- |
| 1 | Frontend gửi câu hỏi với phiên hiện hành; backend xác thực vai trò và xây context chỉ đọc. | Context chỉ gồm dữ liệu được phép xem. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |
| 2 | Backend gửi prompt/context tới FastAPI; FastAPI gọi Gemini và trả câu trả lời/lỗi có kiểm soát. | Không có mutation nghiệp vụ từ chatbot. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |
| 3 | Hiển thị câu trả lời; người dùng đối chiếu trên màn nghiệp vụ khi cần. | Câu trả lời được giới hạn theo role và dữ liệu nguồn. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |

## 4. Luồng thay thế và lỗi

| Mã | Điều kiện | Phản hồi và trạng thái cuối | Test |
| --- | --- | --- | --- |
| A1 | Phiên hết hạn, sai role hoặc context không đủ. | TODO — status/error, rollback và thông báo | TODO |
| A2 | Provider timeout/quota/lỗi hoặc câu trả lời không xác minh được. | TODO — status/error, rollback và thông báo | TODO |
| A3 | Câu hỏi yêu cầu thao tác thanh toán, hoàn tiền, sửa dữ liệu hay tư vấn ngoài phạm vi. | TODO — status/error, rollback và thông báo | TODO |

## 5. Hợp đồng triển khai

| Phần | Cần chốt từ code và yêu cầu |
| --- | --- |
| Frontend | Route, component/state, validation, loading/error, quyền hiển thị |
| API/backend | Method/path, request/response, error, authorization, idempotency |
| Data | Bảng và khóa liên quan, transaction, migration, audit/outbox |
| Tích hợp | Provider/callback, timeout/retry và dữ liệu nhạy cảm (nếu có) |

## 6. Tiêu chí nghiệm thu

- [ ] Member/Coach không nhận dữ liệu của role hoặc chủ thể khác.
- [ ] Lỗi AI không bị trình bày như câu trả lời chắc chắn.
- [ ] Assistant chỉ đọc/hướng dẫn; Manager AI xếp lịch không thuộc use case này.

## 7. Kiểm thử triển khai

- [ ] Happy case được kiểm tra ở UI và API.
- [ ] Các nhánh lỗi/quyền quan trọng có test.
- [ ] Trạng thái sau thao tác khớp UI, API và DB; lưu bằng chứng/PR.

## 8. Truy vết và quyết định còn mở

- [Source of Truth](../requirements/00-Source-of-Truth.md) · [Business Rules](../requirements/SportManagement_BusinessRules_v2.0_updated.docx) · [API contract](../api/api-contract.md) · [Thiết kế hệ thống](../architecture/Center-Management-System-Design-v3.md) · [Demo 6 flow](../operations/LOCAL-DEMO-SCRIPT-6-FLOWS.md)
- **BR / endpoint / màn hình / test liên quan:** TODO.
- **Quyết định còn mở:** TODO.

> Dữ liệu fixture và đường bấm trong tài liệu demo là minh họa, không tự trở thành quy tắc nghiệp vụ.
