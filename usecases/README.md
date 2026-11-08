# Use Cases theo flow

Mỗi flow có **một file Use Case** để nhóm FE/BE/DB cùng triển khai và kiểm thử. Đây là sườn triển khai; các ô `TODO` phải được chốt bằng code và các tài liệu nguồn trước khi đánh dấu hoàn thành.

| Flow | Use Case | Phạm vi chính |
| --- | --- | --- |
| 1 | [UC-F01 — User & Membership](flow-01-user-membership.md) | Người dùng được xác thực/phân quyền đúng; Member có quyền lợi gói Gym hợp lệ và Receptionist kiểm tra được khi check-in. |
| 2 | [UC-F02 — Class booking & Court schedule](flow-02-class-booking-court-schedule.md) | Member chọn lớp hoặc sân khả dụng; hệ thống giữ chỗ, xác nhận quyền sử dụng và hiển thị lịch đúng cho Member/Coach. |
| 3 | [UC-F03 — Payment, wallet & report](flow-03-payment-wallet-report.md) | Hóa đơn, thanh toán VNPay/điểm, quyền lợi và báo cáo tài chính nhất quán. |
| 4 | [UC-F04 — Training & attendance](flow-04-training-attendance.md) | Coach được phân công ghi kế hoạch, điểm danh và kết quả đúng buổi; Member xem dữ liệu của mình. |
| 5 | [UC-F05 — AI workout recommendation](flow-05-ai-workout-recommendation.md) | Member nhận gợi ý tập từ hồ sơ; Coach có thể xem, sửa và duyệt gợi ý trước khi lưu kế hoạch cho học viên được phân công. |
| 6 | [UC-F06 — AI assistant](flow-06-ai-assistant.md) | Assistant trả lời dựa trên ngữ cảnh người đang đăng nhập, không lộ dữ liệu khác vai trò và không tự thực hiện thao tác thay người dùng. |

[Mẫu cho flow mới](TEMPLATE.md)

## Cách dùng khi triển khai

1. Chốt actor, tiền/hậu điều kiện và các nhánh lỗi với Source of Truth/Business Rules.
2. Điền route, endpoint, request/response, quyền, bảng dữ liệu và transaction từ code/API contract.
3. Gắn PR, test và bằng chứng; cập nhật file Use Case khi hành vi thay đổi.
4. Dùng kịch bản demo để đối chiếu thao tác, không dùng fixture làm yêu cầu sản phẩm.
