# Test Strategy

### 2.1 Mục tiêu và ưu tiên

Kiểm thử theo rủi ro, ưu tiên quyền truy cập, tiền/điểm, quyền lợi sau thanh toán và tính đúng của lịch trước các chi tiết trình bày. Mỗi luồng quan trọng cần có happy path, dữ liệu không hợp lệ, thiếu quyền, biên thời gian và thao tác lặp khi phù hợp.

| Mức | Phạm vi ưu tiên | Lý do |
| --- | --- | --- |
| P0 | Đăng nhập, phân quyền/ownership, ví, xác minh thanh toán, cấp quyền lợi, đặt chỗ đồng thời | Sai lệch có thể lộ dữ liệu, sai số dư hoặc cấp dịch vụ sai |
| P1 | Lớp/sân, incident, attendance, hỗ trợ, báo cáo, audit/outbox | Ảnh hưởng vận hành và đối soát |
| P2 | Responsive, nội dung, điều hướng, thông báo trạng thái, trải nghiệm bàn phím | Ảnh hưởng khả năng sử dụng |

P0/P1/P2 ở đây là ưu tiên chạy test, không phải phân loại kết quả hay mức độ lỗi.

### 2.2 Phạm vi kiểm thử

| Flow | Chức năng trong phạm vi |
| --- | --- |
| F01 — User/Membership | Đăng ký/OTP, đăng nhập, quên mật khẩu, trạng thái tài khoản, vai trò, Gym membership/vé ngày, check-in |
| F02 — Class/Schedule | Sport, phòng độc lập, quyền cho thuê, lớp nhiều buổi, Coach, giữ chỗ, ghi danh, ngưỡng mở lớp, lịch sân và incident |
| F03 — Payment/Report | Invoice, checkout, điểm/VNPay, OTP thanh toán hộ, retry, reconciliation, hoàn điểm, hỗ trợ hoàn chuyển khoản và báo cáo |
| F04 — Training/Attendance | Phân công Coach–Member theo lớp, plan/result/homework, điểm danh và quyền xem lịch sử |
| F05 — AI Workout | Gợi ý trong phạm vi học viên được giao, xác thực mục tiêu, kiểm tra output, Coach duyệt trước khi lưu |
| F06 — AI Assistant | Chat/điều hướng Member và Coach, context theo chủ sở hữu, xử lý provider lỗi và giới hạn vai trò |

Kiểm thử xuyên suốt: JWT/security stamp, xác thực dữ liệu phía backend, UTC/giờ Việt Nam, transaction, concurrency, idempotency, audit, notification outbox và giao diện theo vai trò.

### 2.3 Các tầng và phương pháp

| Tầng | Cách thực hiện | Giới hạn |
| --- | --- | --- |
| Unit | xUnit cho rule/service độc lập; thời gian, input biên và tính toán | Không chứng minh database hoặc provider thật |
| Integration | ASP.NET Core test host, PostgreSQL/Testcontainers, assertion HTTP và dữ liệu | Một số dependency được thay bằng test double |
| API contract | Kiểm tra status, schema, mã `error`, quyền và side effect | Không thay thế hành trình UI |
| UI/browser | Playwright; xác định rõ mock route hay live API trong từng suite | Test mock không phải E2E toàn hệ thống |
| AI service | Python tests cho route và lỗi provider; fake model khi được cấu hình | Không xác nhận chất lượng Gemini thật |
| Manual acceptance | 6 flow, giao dịch sandbox, email nhận thực tế, UI và evidence | Chỉ có giá trị cho tình huống và môi trường đã ghi |

Áp dụng equivalence partitioning, boundary value, decision table cho quyền/trạng thái, state transition cho checkout/lớp và concurrency cho tài nguyên hữu hạn. Sau thay đổi, chạy test trực tiếp liên quan rồi regression các flow dùng chung.

[Mục lục tài liệu kiểm thử](README.md)
