# UC-F03 — Payment, wallet & report

> Trạng thái: Draft for implementation · Owner: TODO · Cập nhật: TODO

## 1. Mục tiêu và phạm vi

- **Actor:** Member, Receptionist, Center Manager
- **Mục tiêu:** Hóa đơn, thanh toán VNPay/điểm, quyền lợi và báo cáo tài chính nhất quán.
- **Trigger:** Member hoặc Receptionist bắt đầu checkout; Manager xem đối soát/báo cáo.
- **Ngoài phạm vi:** TODO — ghi rõ chức năng liên quan nhưng không triển khai trong flow này.

## 2. Điều kiện

- **Tiền điều kiện:** TODO — phiên/role, cấu hình, dữ liệu và trạng thái đầu vào.
- **Hậu điều kiện thành công:** TODO — trạng thái nghiệp vụ, dữ liệu, audit và quyền lợi.
- **Hậu điều kiện thất bại:** TODO — rollback, giữ chỗ, thông báo hoặc retry.

## 3. Luồng chính

| Bước | Actor / hệ thống làm gì | Kết quả quan sát được | API / màn hình / bảng liên quan |
| --- | --- | --- | --- |
| 1 | Tạo hóa đơn từ dịch vụ/booking và xác định phần tiền, phần điểm. | Hóa đơn và số tiền/điểm phải trả được hiển thị. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |
| 2 | Xử lý thanh toán; backend xác minh kết quả provider/callback và cập nhật sổ điểm. | Trạng thái thanh toán có bằng chứng, không dựa riêng vào redirect UI. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |
| 3 | Cấp quyền lợi khi hợp lệ; Manager/Receptionist xem hóa đơn, giao dịch và báo cáo. | Hóa đơn, quyền lợi và báo cáo khớp nhau. | TODO — đối chiếu code và [API contract](../api/api-contract.md) |

## 4. Luồng thay thế và lỗi

| Mã | Điều kiện | Phản hồi và trạng thái cuối | Test |
| --- | --- | --- | --- |
| A1 | VNPay trả muộn, thất bại hoặc trạng thái chưa rõ. | TODO — status/error, rollback và thông báo | TODO |
| A2 | Điểm không đủ hoặc hold điểm hết hạn. | TODO — status/error, rollback và thông báo | TODO |
| A3 | Callback lặp, hoàn/hỗ trợ và đối soát sai lệch. | TODO — status/error, rollback và thông báo | TODO |

## 5. Hợp đồng triển khai

| Phần | Cần chốt từ code và yêu cầu |
| --- | --- |
| Frontend | Route, component/state, validation, loading/error, quyền hiển thị |
| API/backend | Method/path, request/response, error, authorization, idempotency |
| Data | Bảng và khóa liên quan, transaction, migration, audit/outbox |
| Tích hợp | Provider/callback, timeout/retry và dữ liệu nhạy cảm (nếu có) |

## 6. Tiêu chí nghiệm thu

- [ ] Callback lặp không cấp quyền lợi hoặc trừ điểm hai lần.
- [ ] Hóa đơn chỉ chuyển trạng thái theo kết quả đã xác minh.
- [ ] Báo cáo và lịch sử ví khớp giao dịch đã ghi nhận.

## 7. Kiểm thử triển khai

- [ ] Happy case được kiểm tra ở UI và API.
- [ ] Các nhánh lỗi/quyền quan trọng có test.
- [ ] Trạng thái sau thao tác khớp UI, API và DB; lưu bằng chứng/PR.

## 8. Truy vết và quyết định còn mở

- [Source of Truth](../requirements/00-Source-of-Truth.md) · [Business Rules](../requirements/SportManagement_BusinessRules_v2.0_updated.docx) · [API contract](../api/api-contract.md) · [Thiết kế hệ thống](../architecture/Center-Management-System-Design-v3.md) · [Demo 6 flow](../operations/LOCAL-DEMO-SCRIPT-6-FLOWS.md)
- **BR / endpoint / màn hình / test liên quan:** TODO.
- **Quyết định còn mở:** TODO.

> Dữ liệu fixture và đường bấm trong tài liệu demo là minh họa, không tự trở thành quy tắc nghiệp vụ.
