# Test Cases

### 4.1 Quy tắc ghi nhận

Danh mục sau là test case ở mức nghiệp vụ, chưa phải toàn bộ test tự động trong repository. ID `TC-*` dùng cho tài liệu này. Mỗi case dưới đây có trạng thái **Not Run cho đợt nghiệm thu mới**, actual result/evidence chưa có. Report lịch sử trong [Test Report](test-report.md) không tự động chuyển các case này thành Pass.

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

[Mục lục tài liệu kiểm thử](README.md)
