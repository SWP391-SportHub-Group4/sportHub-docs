# SportHub — Testing Documentation

Tài liệu kiểm thử được chia thành các file theo từng nội dung. Cơ sở, quy ước chung và kết luận được giữ tại trang này.

## Mục lục

| Nội dung | File / liên kết |
| --- | --- |
| Cơ sở và quy ước | [Đọc tại trang này](#1-cơ-sở-và-quy-ước) |
| Test Strategy | [test-strategy.md](test-strategy.md) |
| Test Plan | [test-plan.md](test-plan.md) |
| Test Cases | [test-cases.md](test-cases.md) |
| Integration Testing | [integration-testing.md](integration-testing.md) |
| API Testing | [api-testing.md](api-testing.md) |
| Test Results / Test Report | [test-report.md](test-report.md) |
| Verification Evidence | [verification-evidence.md](verification-evidence.md) |
| Bug Tracker | [bug-tracker.md](bug-tracker.md) |
| Kết luận | [Đọc tại trang này](#9-kết-luận) |

## 1. Cơ sở và quy ước

### 1.1 Mục đích

Tài liệu xác định hệ thống cần kiểm thử gì, cách tổ chức kiểm thử, kết quả mong đợi, bằng chứng đã có và giới hạn của các kết luận chất lượng. Người đọc phải phân biệt được yêu cầu nghiệp vụ, test đã được viết và test đã thực sự chạy.

### 1.2 Tài liệu đầu vào

| Nguồn | Vai trò trong kiểm thử |
| --- | --- |
| [Source of Truth](../requirements/00-Source-of-Truth.md) | Phạm vi, thuật ngữ, các quyết định nghiệp vụ và thứ tự giải quyết mâu thuẫn |
| [Functional & Nonfunctional Specification](../requirements/Functional-and-Nonfunctional-Spec.md) | Yêu cầu chức năng, bảo mật, dữ liệu, vận hành và giới hạn nghiệm thu |
| [Business Rules](../requirements/SportManagement_BusinessRules_v2.0_updated.docx) | Quy tắc chi tiết; đọc cùng những quyết định thay thế trong Source of Truth |
| [Thiết kế hệ thống](../architecture/Center-Management-System-Design-v3.md) | Transaction, module, background job, rủi ro và tích hợp chưa nghiệm thu |
| [API contract](../api/api-contract.md) | Endpoint, DTO, quyền truy cập và phản hồi lỗi |
| [Use Cases](../usecases/README.md) | Sáu flow nghiệp vụ, luồng chính, nhánh lỗi và truy vết UC |
| [Runbook](../operations/RUNBOOK.md), [Demo 6 flow](../operations/LOCAL-DEMO-SCRIPT-6-FLOWS.md) | Chuẩn bị môi trường và kiểm tra nghiệp vụ thủ công |
| [AI Implementation & Verification](../verification/AI-Member-Coach-Implementation-Verification.md) | Tiêu chí nghiệm thu AI Member/Coach và giới hạn bằng chứng |

Test code và cấu hình được đối chiếu tại repository ứng dụng `sports-center-management-system`. Đường dẫn code trong tài liệu này tính từ gốc repository ứng dụng. Các liên kết tương đối tới tài liệu tính từ thư mục `testing/` của repository tài liệu.

Khi code khác yêu cầu, kết quả mong đợi lấy theo quyết định nghiệp vụ hiện hành. Không đổi expected result chỉ để phù hợp với code. Các phần lịch sử mâu thuẫn phải đọc theo Source of Truth và quyết định cập nhật có hiệu lực.

### 1.3 Quy ước trạng thái

| Trạng thái | Ý nghĩa |
| --- | --- |
| Not Run | Chưa có lần chạy được ghi nhận cho test case trên phiên bản nghiệm thu |
| Pass | Đã chạy và đáp ứng toàn bộ expected result, có evidence |
| Fail | Đã chạy nhưng có kết quả không đúng expected result |
| Blocked | Không thực hiện được do thiếu tiền điều kiện hoặc môi trường |
| Skipped | Runner hoặc người kiểm thử chủ động bỏ qua, cần ghi lý do |
| Historical | Bằng chứng của lần chạy trước; không xác nhận chất lượng phiên bản hiện tại |

Có test code không đồng nghĩa Pass. Build, lint và typecheck thành công không thay thế kiểm thử nghiệp vụ. Không có số liệu coverage không được kết luận đạt 100% coverage.

## 9. Kết luận

Hoạt động kiểm thử của SportHub kết hợp việc thành viên tự kiểm tra các chức năng thuộc phạm vi phụ trách và kiểm thử tự động thông qua GitHub Actions khi tạo hoặc cập nhật pull request vào `main` hoặc `develop`. Kết quả kiểm tra được theo dõi qua PR, commit, log và artifact của workflow.

Tài liệu này xác định chiến lược, phạm vi, các tình huống kiểm thử và tổng hợp bằng chứng hiện có. Các test case và kết quả cần được cập nhật khi chức năng hoặc yêu cầu nghiệp vụ thay đổi.
