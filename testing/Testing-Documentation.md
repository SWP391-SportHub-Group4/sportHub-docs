# SportHub — Testing Documentation

## Mục lục

1. [Cơ sở và quy ước](#1-cơ-sở-và-quy-ước)
2. [Test Strategy](#2-test-strategy)
3. [Test Plan](#3-test-plan)
4. [Test Cases](#4-test-cases)
5. [Integration Testing](#5-integration-testing)
6. [API Testing](#6-api-testing)
7. [Test Results / Test Report](#7-test-results--test-report)
8. [Verification Evidence](#8-verification-evidence)
9. [Kết luận](#9-kết-luận)

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

## 2. Test Strategy

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

## 3. Test Plan

### 3.1 Thông tin đợt kiểm thử

Đợt kiểm thử mới chưa được thực hiện. Các trường dưới đây phải điền khi chạy, không mặc định lấy ngày viết tài liệu làm ngày chạy test.

| Trường | Giá trị hiện tại |
| --- | --- |
| Run ID | Chưa cấp; đề xuất `RUN-YYYYMMDD-NN` |
| Commit, branch, trạng thái working tree | Ghi tại thời điểm chạy; SHA tham chiếu ở đầu tài liệu chưa phải SHA của kết quả lịch sử |
| Người thực hiện/người rà soát | Chưa phân công tên cụ thể |
| Thời gian bắt đầu/kết thúc | Chưa có |
| Môi trường và database | Chưa chọn cho đợt chạy mới |
| Provider mode | Ghi riêng từng dịch vụ: fake/mock/sandbox/live |

### 3.2 Môi trường và dữ liệu

- Backend: SDK/runtime .NET 10; năm test project Security, Scheduling, Payment, Administration và Training dùng xUnit.
- Database: PostgreSQL; CI chuẩn bị image `postgres:16-alpine`. Đọc factory của từng suite để xác định tạo database/container và vòng đời dữ liệu trước khi chạy.
- Frontend: Node `>=24 <25`, npm `>=11 <12` theo `frontend/package.json`; cài dependency bằng lockfile. Playwright dùng Desktop Chrome/Chromium và bản Next.js đã build.
- Cấu hình Playwright mặc định: frontend `http://127.0.0.1:3100`, API `http://127.0.0.1:5000`; có thể override bằng biến môi trường. Tránh dùng nhầm server đang chạy với dữ liệu khác.
- Python AI: dùng môi trường tách biệt và dependency của `ai-service`; xác định fake model hay provider thật trước khi diễn giải kết quả.
- Dùng database test độc lập, dữ liệu giả, không dùng tài khoản hoặc dữ liệu cá nhân thật. Không trỏ suite có seed/mutation vào database vận hành.
- Dữ liệu gồm năm role và Guest; tài khoản Active/Deactivated; lớp Draft/Published/InProgress/Completed/Cancelled; lớp còn chỗ/hết chỗ; phòng active/inactive và cho thuê/tắt thuê; ví đủ/thiếu điểm; invoice và attempt ở các trạng thái hợp lệ.
- Kiểm tra mốc giờ Việt Nam, sát thời điểm bắt đầu/kết thúc, cuối tháng/năm nhuận; snapshot giá và dữ liệu trước/sau thao tác.

### 3.3 Phân công kiểm thử theo phạm vi phụ trách

Nhóm thực hiện kiểm thử chủ yếu theo phạm vi chức năng được giao. Thành viên phụ trách phát triển các chức năng của vai trò nào sẽ trực tiếp kiểm tra các chức năng thuộc vai trò đó.

| Phạm vi phụ trách | Trách nhiệm kiểm thử |
| --- | --- |
| Member | Kiểm tra các chức năng dành cho hội viên và quyền truy cập tương ứng |
| Receptionist | Kiểm tra các nghiệp vụ tại quầy và quyền thao tác của lễ tân |
| Coach | Kiểm tra các chức năng huấn luyện, lớp được phân công và quyền truy cập học viên |
| Center Manager | Kiểm tra các chức năng quản lý và vận hành trung tâm |
| System Administrator | Kiểm tra quản trị tài khoản, phân quyền và nhật ký thao tác |

Khi phát hiện lỗi trong phần phụ trách, thành viên chỉnh sửa và kiểm tra lại. Với các luồng liên quan đến nhiều vai trò, các thành viên phụ trách cần phối hợp kiểm tra tính nhất quán giữa các bước. Yêu cầu phối hợp này không xác nhận nhóm đã hoàn thành kiểm thử tích hợp cho mọi luồng.

### 3.4 Trình tự thực hiện

Nhóm thực hiện kiểm thử tự động thông qua GitHub Actions của repository ứng dụng. Nhóm không tự đặt hoặc ghi Run ID riêng; pull request, commit và liên kết lần chạy GitHub Actions được dùng để xác định phiên bản và kết quả kiểm thử. GitHub tự cấp mã cho mỗi lần chạy workflow.

1. Thành viên hoàn thành thay đổi trong phạm vi phụ trách, commit và push code lên nhánh làm việc trên GitHub.
2. Tạo pull request vào `main` hoặc `develop`. Theo cấu hình `.github/workflows/ci.yml` hiện có, GitHub Actions tự kích hoạt workflow cho PR vào hai nhánh này; push trực tiếp lên hai nhánh cũng kích hoạt workflow.
3. GitHub Actions chuẩn bị môi trường và chạy các bước đã cấu hình, gồm kiểm tra deployment scripts, build/test backend, kiểm tra frontend và job E2E. Job E2E phụ thuộc kết quả backend/frontend, nên có thể không chạy nếu các job trước không đạt. Workflow chỉ kiểm tra những tình huống đã được cấu hình và viết test.
4. Thành viên xem tiến trình và kết quả trong phần **Checks** của PR hoặc tab **Actions**; khi cần, mở log và tải artifact kiểm thử được workflow lưu.
5. Nếu kiểm tra không đạt, thành viên phụ trách xem log, xác định nguyên nhân, chỉnh sửa rồi push commit mới lên cùng nhánh PR. GitHub Actions tự chạy lại với thay đổi mới; tiếp tục kiểm tra kết quả của commit mới nhất.
6. Khi tổng hợp Test Report, tham chiếu PR, commit và liên kết lần chạy Actions cùng log/artifact hiện có. Phân biệt test Pass/Fail với job bị lỗi môi trường, bị bỏ qua hoặc bị hủy; không coi workflow chưa hoàn tất là đã Pass.

Kiểm tra thủ công do thành viên phụ trách chức năng thực hiện như mục 3.3. GitHub Actions không tự chứng minh các thao tác thủ công hoặc tích hợp dịch vụ thật đã được nghiệm thu. Các mẫu Run ID ở những phần khác là đề xuất cho việc ghi nhận bổ sung; với kiểm thử qua CI của nhóm, có thể dùng liên kết lần chạy GitHub Actions thay cho mã tự đặt.

### 3.5 Điều kiện bắt đầu và kết thúc kiểm thử

**Điều kiện bắt đầu:** Thành viên push thay đổi lên GitHub và tạo hoặc cập nhật PR vào `main` hoặc `develop`. GitHub Actions được kích hoạt để chuẩn bị môi trường và chạy các bước kiểm tra theo cấu hình workflow.

**Điều kiện hoàn tất kiểm tra tự động cho PR:** Các job kiểm tra được yêu cầu trong workflow hoàn thành thành công trên commit mới nhất của PR. Nếu có bước thất bại, thành viên phụ trách kiểm tra nguyên nhân, chỉnh sửa và push lại để chạy kiểm tra mới. Các job bị bỏ qua, bị hủy hoặc không chạy được cần được xem xét, không tính là kiểm thử đã Pass.

## 4. Test Cases

### 4.1 Quy tắc ghi nhận

Danh mục sau là test case ở mức nghiệp vụ, chưa phải toàn bộ test tự động trong repository. ID `TC-*` dùng cho tài liệu này. Mỗi case dưới đây có trạng thái **Not Run cho đợt nghiệm thu mới**, actual result/evidence chưa có. Report lịch sử ở mục 7 không tự động chuyển các case này thành Pass.

Khi thực hiện, ghi thêm Run ID, người chạy, thời điểm, actual result, status và evidence path cho từng case. Các dòng gộp nhiều biến thể phải ghi kết quả từng biến thể; chỉ Pass khi mọi biến thể được chọn đều đạt.

### 4.2 F01 — User và Membership

| ID / Ưu tiên | Điều kiện và dữ liệu | Bước thực hiện | Kết quả mong đợi |
| --- | --- | --- | --- |
| TC-F01-01 / P0 | Email chưa dùng; OTP test có thể nhận | Đăng ký; xác nhận OTP hợp lệ; thử dùng lại OTP | Kích hoạt đúng tài khoản; OTP đã dùng không được tái sử dụng |
| TC-F01-02 / P0 | Tài khoản Active, mật khẩu hợp lệ | Đăng nhập; gọi API cần xác thực | Login trả 200, token/profile đúng; API chấp nhận đúng quyền |
| TC-F01-03 / P0 | Email không tồn tại hoặc sai mật khẩu | Gửi lần lượt các input sai và DTO thiếu/sai định dạng | Sai thông tin trả 401, không lộ trạng thái tài khoản; DTO không hợp lệ trả 400 theo contract |
| TC-F01-04 / P0 | Tài khoản có token hợp lệ | Deactivate tài khoản hoặc đổi mật khẩu/role; dùng lại token cũ | Request tiếp theo từ chối token không còn hợp lệ; không vượt quyền do token cũ |
| TC-F01-05 / P0 | Active Admin cuối cùng; Admin đang đăng nhập | Thử tự Deactivate và Deactivate Admin Active cuối cùng | Bị chặn; không thay đổi sai trạng thái/audit |
| TC-F01-06 / P1 | Vé Gym ngày; clock kiểm soát | Mua lúc 05:59:59, 06:00, 21:59:59, 22:00 giờ Việt Nam; kiểm tra hạn | Chỉ bán từ 06:00 đến trước 22:00; vé hết hạn 22:00 cùng ngày, không kéo dài 24 giờ |
| TC-F01-07 / P1 | Member có quyền Gym; Receptionist | Check-in hợp lệ; lặp trong 10 giây; thử vé hết hạn | Check-in đúng quyền lợi; lặp trong cooldown bị chặn; hết hạn không được vào |

### 4.3 F02 — Class, Schedule và Court

| ID / Ưu tiên | Điều kiện và dữ liệu | Bước thực hiện | Kết quả mong đợi |
| --- | --- | --- | --- |
| TC-F02-01 / P1 | Manager; Sport cho thuê hợp lệ | Tạo phòng cùng giá/lịch thuê; lặp với Sport thiếu/không hợp lệ | Hợp lệ lưu toàn bộ; input sai bị từ chối, không tạo dữ liệu dở dang |
| TC-F02-02 / P0 | Lớp capacity 1; hai Member | Gửi đồng thời hai yêu cầu giữ chỗ | Chỉ một thành công; yêu cầu còn lại `class_full`; ReservedCount và active hold bằng 1 |
| TC-F02-03 / P0 | Phòng hoặc Coach đã có lịch | Tạo lịch trùng; tạo lịch mới bắt đầu đúng lúc lịch cũ kết thúc | Lịch trùng bị chặn; tiếp giáp được xét theo `[start,end)` nếu các điều kiện khác hợp lệ |
| TC-F02-04 / P1 | Lớp InProgress còn chỗ, mở đăng ký, còn buổi tương lai | Member mua cả khóa giữa chừng | Tính giá trọn khóa; tham gia từ buổi kế tiếp; không tạo vắng mặt cho buổi đã qua |
| TC-F02-05 / P1 | Lớp có đợt đánh giá ngưỡng và clock test | Xét đạt ngưỡng; xét hết hạn thiếu ngưỡng không được miễn | Đạt ngưỡng xác nhận đúng người còn ghi danh; thiếu ngưỡng xử lý hủy/hoàn theo rule; không tự khởi giảng khi chưa chốt |
| TC-F02-06 / P1 | Incident hợp lệ, fence còn hạn, buổi bị ảnh hưởng | Manager bố trí buổi bù đủ thời lượng; thử lịch xung đột/nằm trong incident | Chỉ lịch hợp lệ được lưu; không xung đột, giữ lịch sử và toàn vẹn transaction |
| TC-F02-07 / P1 | Phòng có lượt thuê và lớp hiện hữu | Tắt Allow rental; thử đặt lượt thuê mới | Không nhận thuê mới; không âm thầm xóa lớp/lượt thuê đã tồn tại |

### 4.4 F03 — Payment, Wallet, Support và Report

| ID / Ưu tiên | Điều kiện và dữ liệu | Bước thực hiện | Kết quả mong đợi |
| --- | --- | --- | --- |
| TC-F03-01 / P0 | Member có đủ điểm, dịch vụ hợp lệ | Checkout hoàn toàn bằng điểm; gửi lại cùng idempotency key | Invoice/ledger/quyền lợi nhất quán; không trừ hoặc cấp trùng |
| TC-F03-02 / P0 | Checkout có tiền qua gateway | Gửi callback hợp lệ; gửi lặp; gửi sai amount/checksum/reference | Chỉ dữ liệu xác minh hợp lệ mới ghi Payment/cấp quyền lợi; callback lặp không tạo thêm bản ghi |
| TC-F03-03 / P0 | PaymentAttempt đang Pending | Mở return page hoặc sửa query hiển thị | Không cấp dịch vụ chỉ từ thông tin hiển thị chưa được server xác minh; UI đọc trạng thái backend |
| TC-F03-04 / P0 | Receptionist thanh toán dùng điểm hộ Member | Thử thiếu/sai/hết hạn OTP; sau đó OTP hợp lệ | Không giữ/trừ điểm khi chưa được xác nhận; xác nhận đúng Member rồi mới thực hiện |
| TC-F03-05 / P0 | Top-up; callback test kiểm soát | Thử dưới 10.000, trên 10.000.000 VND, không bội số 1.000; thanh toán hợp lệ và callback lặp | Input sai bị chặn; đúng số điểm 1 điểm = 1.000 VND; cộng một lần |
| TC-F03-06 / P0 | Thanh toán đến sau hết hold | Reconcile khi còn chỗ; lặp khi đã hết chỗ | Còn chỗ: cấp một lần, PAID_AFTER_RECONCILIATION; hết chỗ: không vượt capacity, đi vào lựa chọn bồi hoàn/hỗ trợ |
| TC-F03-07 / P1 | Invoice đã thanh toán | Member tự hủy dịch vụ; riêng tình huống trung tâm hủy hợp lệ | Tự hủy không tự hoàn; trung tâm hủy hoàn theo item/cap do backend tính; không hoàn trùng |
| TC-F03-08 / P1 | Member có invoice; PAYMENT Support Request | Gửi thiếu invoice/ảnh; gửi hợp lệ với 1–3 ảnh; Receptionist thử ghi nhận payout | Validate đúng; không yêu cầu tài khoản ngân hàng; chỉ Manager được ghi nhận payout với chứng từ theo contract |
| TC-F03-09 / P1 | Dữ liệu giao dịch biết trước, bao gồm hoàn và điểm | Xem báo cáo theo kỳ Việt Nam; đối chiếu export và ledger | Đúng kỳ/snapshot; không coi điểm hoàn là doanh thu tiền mới; export thống nhất với số liệu cùng bộ lọc |

### 4.5 F04 — Training và Attendance

| ID / Ưu tiên | Điều kiện và dữ liệu | Bước thực hiện | Kết quả mong đợi |
| --- | --- | --- | --- |
| TC-F04-01 / P0 | Coach chủ nhiệm A, Coach không liên quan B | A tạo plan/result cho học viên được giao; B thử đọc/ghi | A được thao tác trong phạm vi; B bị từ chối, không lộ dữ liệu |
| TC-F04-02 / P1 | Member chung hai lớp với một Coach | Kết thúc ghi danh lần lượt từng lớp; sau đó học lại | Quan hệ Active khi còn lớp hợp lệ; hết lớp chung thì kết thúc; học lại tạo bản ghi mới, giữ lịch sử |
| TC-F04-03 / P0 | Coach dạy thay một buổi | Đọc/điểm danh buổi được giao; thử đọc hồ sơ/nội dung toàn khóa | Chỉ có quyền của buổi; không tự mở quyền hồ sơ đầy đủ |
| TC-F04-04 / P1 | Lớp Completed; Member còn Confirmed, EndedAt null | Xem nội dung chung và dữ liệu cá nhân; thử đọc dữ liệu người khác | Đọc lịch sử theo rule; không truy cập dữ liệu cá nhân Member khác |
| TC-F04-05 / P1 | Coach có buổi Scheduled đủ điều kiện báo vắng | Gửi ABSENCE và rút Pending; thử gửi loại đề xuất cũ qua API | Hành vi mong đợi: Coach chỉ tạo ABSENCE, rút Pending đúng quyền; loại cũ phải bị chặn khi tạo mới. Cần xác minh lại gap COACH-01 |

### 4.6 F05/F06 — AI

| ID / Ưu tiên | Điều kiện và dữ liệu | Bước thực hiện | Kết quả mong đợi |
| --- | --- | --- | --- |
| TC-F05-01 / P1 | Coach có học viên hợp lệ | Gửi mục tiêu rõ, mơ hồ nhưng liên quan, vô nghĩa hoặc ngoài môn | Phân biệt VALID/VAGUE/INVALID theo policy; input INVALID không gọi generation |
| TC-F05-02 / P0 | Fake provider có timeout hoặc JSON sai | Yêu cầu tạo gợi ý; thử output set/rep không hợp lệ | Không bịa kế hoạch; phản hồi lỗi phù hợp; output phải qua kiểm tra; chỉ lưu khi Coach duyệt |
| TC-F06-01 / P0 | Member và Coach có context riêng | Hỏi về dữ liệu cá nhân; thử yêu cầu dữ liệu người khác hoặc thao tác staff | Không lộ context ngoài quyền; không mạo danh vai trò hoặc thực hiện workflow staff |
| TC-F06-02 / P1 | Bản đồ điều hướng đã đối chiếu frontend | Hỏi vị trí invoices, Personal Plan, account bằng EN/VI | Đúng route và nhãn UI theo role; không bịa bước; thừa nhận khi chưa có hướng dẫn đã xác minh |
| TC-F06-03 / P1 | Prompt/version và context có thể thay đổi | Tiếp tục hội thoại sau đổi prompt/quyền; thử prompt injection | Revalidate continuation theo policy; không vượt scope hoặc bỏ qua kiểm tra quyền |

### 4.7 Kiểm thử xuyên suốt

| ID / Ưu tiên | Điều kiện và bước | Kết quả mong đợi |
| --- | --- | --- |
| TC-X-01 / P0 | Gửi request không token, token hết hạn, sai role và ID thuộc người khác | 401/403 theo contract; không thay đổi dữ liệu hoặc lộ thông tin ngoài scope |
| TC-X-02 / P1 | Giả lập lỗi trong bước ghi dữ liệu/audit/outbox, chạy lại dispatcher | Rollback đúng transaction; dispatch sau commit; retry không tạo thông báo/nghiệp vụ trùng |
| TC-X-03 / P2 | Kiểm tra màn quan trọng ở 320, 1366, 1440 px, dùng bàn phím và reduced motion | Thao tác chính truy cập được, không mất nội dung; focus và trạng thái loading/error rõ |

## 5. Integration Testing

### 5.1 Ma trận tích hợp

Các file sau đã được tìm thấy; danh sách là điểm truy vết test code, không phải xác nhận kết quả chạy mới.

| ID | Thành phần và tình huống | Assertion trọng tâm | Test code tham chiếu |
| --- | --- | --- | --- |
| IT-01 | Identity → JWT → database auth state | Login đúng; token cũ mất hiệu lực khi tài khoản/quyền thay đổi | `backend/SportHub.Security.Tests/Integration/LoginContractTests.cs`, `AccountStatusJwtTests.cs` |
| IT-02 | Scheduling → hold → PostgreSQL transaction | Không bán vượt; không trùng tài nguyên/Member | `backend/SportHub.Scheduling.Tests/Integration/SeatHoldConcurrencyTests.cs`, `OccupancyConcurrencyTests.cs` |
| IT-03 | Checkout → payment → wallet → membership/enrollment/rental | Amount đúng; cấp một lần; dữ liệu rollback nhất quán | `backend/SportHub.Payment.Tests/Integration/FullPaymentAndActivationTests.cs`, `CheckoutFlowTests.cs`, `WalletConcurrencyTests.cs` |
| IT-04 | Gateway event → verification → reconciliation | Chống giả mạo/replay; xử lý thanh toán muộn | `backend/SportHub.Payment.Tests/Integration/PaymentReturnVerificationTests.cs`, `FrontDeskPaymentReconciliationTests.cs` |
| IT-05 | Incident → schedule → refund/notice | Fence, cập nhật lịch và quyền lợi nhất quán | `backend/SportHub.Scheduling.Tests/Integration/IncidentSessionRecoveryTests.cs`, `backend/SportHub.Payment.Tests/Integration/IncidentRefundTests.cs` |
| IT-06 | Training → enrollment → authorization | Kết thúc/duy trì quan hệ đúng; quyền lịch sử và phạm vi Coach | `backend/SportHub.Training.Tests/Integration/RelationshipLifecycleTests.cs`, `backend/SportHub.Security.Tests/Integration/CoachMemberReadScopeTests.cs` |
| IT-07 | Business transaction → outbox → dispatcher | Tạo event cùng commit; retry/dedup | `backend/SportHub.Payment.Tests/Integration/OutboxDispatchTests.cs` |
| IT-08 | Ledger/invoice → report/export | Kỳ báo cáo, snapshot và đối soát thống nhất | `backend/SportHub.Payment.Tests/Integration/ReportReconciliationTests.cs`, `ReportExportParityTests.cs` |
| IT-09 | AI member/coach routes → fake model | Tách wrapper theo vai trò; route không tồn tại bị từ chối | `ai-service/tests/test_role_routes.py` |

### 5.2 Quy trình và giới hạn

1. Seed dữ liệu với ID riêng; ghi số dư, số chỗ, trạng thái và số bản ghi trước thao tác.
2. Thực hiện qua HTTP hoặc service đúng cấp độ mà test đang kiểm tra; không gọi service rồi mô tả như đã đi qua toàn bộ HTTP middleware.
3. Với concurrency, gửi các thao tác đồng thời, kiểm tra cả response và trạng thái cuối trong database.
4. Với retry, gửi lại request/callback và kiểm tra Payment, ledger, quyền lợi, audit/outbox không phát sinh sai.
5. Lưu result và tên test cụ thể; teardown theo vòng đời fixture, không xóa database dùng chung.

Chạy integration với PostgreSQL thật chứng minh hành vi transaction trong cấu hình test; không tự chứng minh mạng public, provider thật, triển khai production hoặc E2E toàn hệ thống.

## 6. API Testing

### 6.1 Phương pháp

Đối chiếu API contract với controller/OpenAPI của phiên bản chạy. Mỗi test ghi method/URL, actor, header không chứa secret, payload, expected status/body, actual response và side effect. Với API mutate, đọc lại trạng thái hoặc database để kiểm tra kết quả.

| Nhóm API | Kiểm tra chính | Nguồn/case |
| --- | --- | --- |
| `POST /api/auth/login` | Valid trả 200; sai thông tin 401; invalid DTO 400; Deactivated bị chặn theo contract; không lộ password/hash | `LoginContractTests.cs`; TC-F01-02/03/04 |
| `POST /api/rooms` | Manager-only; rentalSetup hợp lệ; input sai không để phòng dở dang | TC-F02-01; API contract phần phòng độc lập |
| `PUT /api/rooms/{roomId}/rental-access` | CenterManager-only; bật/tắt quyền thuê; bảo toàn booking/lớp cũ | TC-F02-07 |
| `POST /api/checkouts/membership` | Đúng Member/package, giá backend, idempotency key; test hiện có kỳ vọng 201 khi tạo mới | `PaymentReturnVerificationTests.cs`; TC-F03-01/02 |
| `POST /api/checkouts/{invoiceId}/attempts` | Invoice đúng chủ thể/trạng thái; tạo attempt đúng amount; không cấp quyền từ URL đơn thuần | TC-F03-02/03 |
| `POST /api/checkouts/verify-reference` | Verify signed fields; sửa amount bị từ chối, Payment không phát sinh | `PaymentReturnVerificationTests.cs`; TC-F03-02 |
| `POST /api/support/requests` | Ownership invoice, ảnh/validation, scope xử lý; kiểm tra schema theo contract hiện hành | TC-F03-08 |
| `GET /api/manager/operations/health` | Quyền Manager; trạng thái job/outbox/reconciliation theo dữ liệu | `OperationsHealthTests.cs`; đặc tả vận hành |
| Các API training/AI | Role kết hợp quan hệ được giao; chặn ID ngoài scope, input không hợp lệ, provider failure | TC-F04-01/03, TC-F05-02, TC-F06-01 |

Status nêu cho một endpoint không áp dụng cứng cho mọi endpoint khác. Đặc biệt không mặc định mọi input nghiệp vụ sai đều là 400: 409 và mã `error` phải theo contract/test cụ thể.

### 6.2 Các chiều kiểm tra bắt buộc theo endpoint

- Authentication: thiếu token, token sai/hết hạn hoặc bị vô hiệu sau đổi account state.
- Authorization: đúng role nhưng sai owner/assignment; không chỉ kiểm tra menu frontend.
- Validation: thiếu trường, null/blank, sai type, biên độ dài/số tiền/thời gian; xác nhận không có partial write.
- Response contract: status, trường bắt buộc, kiểu dữ liệu, mã lỗi ổn định và dữ liệu không được phép trả.
- Mutation: kiểm tra snapshot giá, số dư/ledger, trạng thái invoice và quyền lợi trước/sau.
- Retry/concurrency: idempotency key, callback replay, thay đổi dữ liệu giữa lúc đọc và ghi.
- Privacy: không đưa JWT, OTP, password, hash, API key hay PII vào evidence công khai.

Không bắt buộc dùng Postman: HTTP integration test có assertion và report cũng là API testing evidence. Nếu xuất collection, chỉ lưu biến mẫu và dữ liệu giả.

## 7. Test Results / Test Report

### 7.1 Kết quả lần biên soạn 18/11/2026

| Hoạt động | Kết quả |
| --- | --- |
| Đọc yêu cầu, use cases, API, verification và cấu hình/test code | Đã thực hiện để xây dựng tài liệu |
| Chạy lại unit/integration/API/Playwright/AI tests | Chưa thực hiện trong lần biên soạn |
| Chứng nhận Pass trên phiên bản hiện tại | Chưa có cơ sở |
| Xác nhận độ bao phủ phần trăm | Chưa có báo cáo coverage được thẩm định |

### 7.2 Kết quả backend lịch sử

Nguồn: `backend/ci-local-test.log` trong workspace ứng dụng. Tên TRX được log dẫn tới có dấu thời gian `20261104`; ngày 04/11/2026 được suy ra từ tên file, chưa chuẩn hóa timezone hay xác minh thời điểm từ nội dung TRX. Chưa xác định commit của lần chạy. Số liệu sau được chép từ phần tổng kết log, không phải kết quả chạy mới.

| Test project | Passed | Failed | Skipped | Total | Thời lượng theo log |
| --- | ---: | ---: | ---: | ---: | --- |
| SportHub.Administration.Tests | 29 | 0 | 0 | 29 | 4 giây |
| SportHub.Training.Tests | 12 | 0 | 0 | 12 | 10 giây |
| SportHub.Security.Tests | 308 | 0 | 0 | 308 | 2 phút 7 giây |
| SportHub.Payment.Tests | 382 | 0 | 1 | 383 | 3 phút 57 giây |
| SportHub.Scheduling.Tests | 341 | 0 | 0 | 341 | 4 phút 51 giây |
| **Tổng** | **1.072** | **0** | **1** | **1.073** | Không cộng thời lượng để suy ra elapsed time |

Lý do và tên test bị skipped chưa được xác minh từ log tổng kết. Cần đọc TRX tương ứng trước khi kết luận phạm vi bỏ qua. Không diễn giải 1.072 test Pass thành bao phủ 1.072 yêu cầu hoặc toàn bộ test hiện có.

### 7.3 Kết quả frontend lịch sử

Nguồn: `frontend/ci-browser-test.log`; log ghi `Running 42 tests using 1 worker` và `42 passed (2.0m)`. Các dòng đã xem gồm loading, public/auth và reports dashboard; không dùng kết quả này làm tổng kết toàn bộ thư mục `frontend/tests/` hiện tại. Chưa xác định thời điểm, commit, danh sách dependency và mode API đầy đủ của lần chạy.

| Passed | Failed được báo ở tổng kết | Phạm vi kết luận |
| ---: | --- | --- |
| 42 | Không thấy failure trong tổng kết đã đọc | Chỉ 42 trường hợp được log ghi nhận; bằng chứng lịch sử |

## 8. Verification Evidence

### 8.1 Danh mục nguồn hiện có

| Evidence ID | Nguồn | Chứng minh được | Giới hạn |
| --- | --- | --- | --- |
| EV-01 | `backend/ci-local-test.log` | Tổng kết năm test project lịch sử tại mục 7.2 | Chưa gắn commit; chưa kiểm tra nội dung các TRX được dẫn |
| EV-02 | `frontend/ci-browser-test.log` | 42 test được log báo Pass | Chưa có metadata đủ để tái lập; không đại diện toàn bộ suite hiện tại |
| EV-03 | `.github/workflows/ci.yml` | CI có bước build/test, chuẩn bị PostgreSQL, upload TRX và job E2E | Cấu hình workflow không chứng minh job đã chạy thành công |
| EV-04 | `frontend/playwright.config.ts` | Cấu hình trace/screenshot/video và web server | Artifact chỉ tồn tại khi thực sự chạy và điều kiện lưu được kích hoạt |
| EV-05 | Test code được dẫn ở mục 5/6 | Có assertion và tình huống đã được triển khai | Sự tồn tại của file không chứng minh Pass |
| EV-06 | Tài liệu AI Verification | Tiêu chí nghiệm thu, scope và giới hạn AI | Không phải report runtime của phiên bản hiện tại |

EV-01 đến EV-05 đang nằm trong repository ứng dụng tại workspace, chưa được đóng gói thành bộ evidence trong repository tài liệu. Các file log local có thể không được Git theo dõi; khi bàn giao phải đính kèm bản đã rà soát hoặc artifact CI có thể truy cập. Không coi đường dẫn local là bằng chứng đã được chia sẻ cho cả nhóm.

## 9. Kết luận

Hoạt động kiểm thử của SportHub kết hợp việc thành viên tự kiểm tra các chức năng thuộc phạm vi phụ trách và kiểm thử tự động thông qua GitHub Actions khi tạo hoặc cập nhật pull request vào `main` hoặc `develop`. Kết quả kiểm tra được theo dõi qua PR, commit, log và artifact của workflow.

Tài liệu này xác định chiến lược, phạm vi, các tình huống kiểm thử và tổng hợp bằng chứng hiện có. Các test case và kết quả cần được cập nhật khi chức năng hoặc yêu cầu nghiệp vụ thay đổi.
