# Use Cases theo 6 flow của SRS

Mỗi flow có **một file Use Case** mô tả mục tiêu, actor, điều kiện, luồng chính, nhánh lỗi, hậu điều kiện, nghiệm thu và mã UC/BR truy vết. Nguồn nền là *SportHub SRS Part I*, §2 và §4.2 (UC-01–76). Các quyết định mới hơn trong [Source of Truth](../requirements/00-Source-of-Truth.md), [Business Rules](../requirements/SportManagement_BusinessRules_v2.0_updated.docx), [thiết kế](../architecture/Center-Management-System-Design-v3.md) và [API contract](../api/api-contract.md) được ưu tiên khi SRS cũ mâu thuẫn. Đây là yêu cầu và luồng nghiệm thu, không tự khẳng định mọi nhánh đã được triển khai.

| Flow | Use Case | Phạm vi chính |
| --- | --- | --- |
| 1 | [UC-F01 — Tài khoản và Membership Gym](flow-01-user-membership.md) | UC-01–15, UC-22–27: danh tính, staff, catalog, quyền Gym và check-in. |
| 2 | [UC-F02 — Lớp học, giữ chỗ và lịch sân](flow-02-class-booking-court-schedule.md) | UC-16–21, UC-28–46; UC-76 phụ trợ: lớp, phòng, sân, ngưỡng và sự cố. |
| 3 | [UC-F03 — Hóa đơn, ví điểm, thanh toán và báo cáo](flow-03-payment-wallet-report.md) | UC-47–65: checkout, VNPay, hoàn, hỗ trợ, báo cáo và thông báo. |
| 4 | [UC-F04 — Giảng dạy, điểm danh và kết quả](flow-04-training-attendance.md) | UC-66–73: dữ liệu lớp/học viên, attendance, plan và kết quả. |
| 5 | [UC-F05 — Gợi ý bài tập bằng AI](flow-05-ai-workout-recommendation.md) | UC-74: gợi ý cho Member/Coach, Coach duyệt trước khi lưu. |
| 6 | [UC-F06 — Trợ lý AI theo ngữ cảnh](flow-06-ai-assistant.md) | UC-75: chat chỉ đọc cho Member/Coach; không cấp quyền Manager. |

[Mẫu cho flow mới](TEMPLATE.md)

## Cách dùng khi triển khai và nghiệm thu

1. Đọc file flow cùng mã UC/BR tương ứng; dùng API contract để lấy route/DTO chính xác ở phiên bản code đang nghiệm thu.
2. Đối chiếu cả luồng chính và nhánh lỗi với UI, API, database, audit và test. Ghi gap khi yêu cầu chưa có trong code.
3. Khi hành vi thay đổi, cập nhật UC và tài liệu nguồn liên quan. Dữ liệu trong [kịch bản demo](../operations/LOCAL-DEMO-SCRIPT-6-FLOWS.md) chỉ là fixture minh họa.
