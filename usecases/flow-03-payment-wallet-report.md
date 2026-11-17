# UC-F03 — Hóa đơn, ví điểm, thanh toán và báo cáo

> Căn cứ: *SportHub SRS Part I* §2.3, §4.2 UC-47–65. Áp dụng cập nhật thanh toán/hỗ trợ trong [Source of Truth](../requirements/00-Source-of-Truth.md), [Business Rules](../requirements/SportManagement_BusinessRules_v2.0_updated.docx), [thiết kế hệ thống](../architecture/Center-Management-System-Design-v3.md) và [API contract](../api/api-contract.md). Flow 3 là phạm vi bắt buộc.

## Mục tiêu và actor

Member hoặc Receptionist tạo checkout cho đúng chủ thể; hệ thống thu điểm/VNPay, xác minh trước khi cấp Membership, Enrollment hoặc rental; CenterManager duyệt ngoại lệ, xử lý khoản hoàn chuyển khoản qua Support Request và xem báo cáo. VNPay, job đối soát, job outbox là actor hệ thống. Tạo quyền lợi bắt nguồn từ [Flow 1](flow-01-user-membership.md) và [Flow 2](flow-02-class-booking-court-schedule.md).

| UC SRS | Phạm vi | Kết quả |
| --- | --- | --- |
| UC-47–50 | Checkout, xác nhận điểm tại quầy, VNPay, invoice | Thu đúng số, cấp dịch vụ sau xác minh, không cấp trùng |
| UC-51–55 | Ví, nạp điểm, refund, lựa chọn hoàn | Ledger và quyền lợi nhất quán |
| UC-56–60 | Support Request, SLA/remind, payout, xác nhận nhận tiền | Lịch sử xử lý và chứng từ rõ, không lưu số tài khoản |
| UC-61–65 | Báo cáo, export, report lifecycle, notification | Doanh thu/điểm đúng kỳ; thông báo sau commit |

## Tiền điều kiện và bất biến

- Backend tính giá theo snapshot; client không quyết định amount. `TotalAmount = PointsApplied × 1.000 + CashAmount`, 1 điểm = 1.000 VND. Hóa đơn đã Paid không sửa hoặc void. Mỗi checkout/attempt có mã định danh và idempotency; retry không tạo Payment/quyền lợi/hoàn điểm trùng.
- Seat/slot, quyền dùng điểm và điều kiện cấp dịch vụ được kiểm lại trong transaction. Điểm dùng hộ ở quầy cần OTP Member trước khi giữ/trừ. Return URL chỉ phục vụ hiển thị, không xác nhận tiền; chỉ IPN/QueryDR đã kiểm checksum, TmnCode, TxnRef, amount và mã phản hồi mới được ghi Payment.
- Ledger điểm là lịch sử bất biến. Hoàn điểm theo invoice item, giới hạn do server tính; Member tự hủy dịch vụ đã mua không được hoàn. Khoản hoàn chuyển khoản do Manager thực hiện ngoài hệ thống và ghi nhận qua Support Request, không gọi VNPay Refund API.
- Báo cáo tách tiền VNPay thực thu, điểm dùng, điểm phát hành/hoàn, tiền hoàn chuyển khoản đã ghi nhận và dịch vụ được cấp. Điểm hoàn không phải doanh thu tiền mới; kỳ báo cáo theo ngày giờ Việt Nam.

## Luồng chính

### F03-A — Checkout và xác minh (UC-47–50)

1. Member hoặc Receptionist đã chọn đúng Member bắt đầu mua một mục dịch vụ. Backend lấy giá/quyền sở hữu và điều kiện hiện hành, tạo invoice item snapshot, checkout có hạn và resource hold nếu cần. UI hiện tổng, điểm dùng, tiền còn lại và `expiresAtUtc` từ server; reload không đặt lại đồng hồ.
2. Member chọn số điểm. Nếu Receptionist dùng điểm hộ, Member xác nhận OTP 6 số qua email trước khi điểm được giữ/trừ. Nếu tiền còn lại bằng 0, hệ thống xác nhận trong transaction; nếu có tiền mặt, backend tạo PaymentAttempt và URL VNPay đã ký.
3. VNPay gửi IPN hoặc backend QueryDR; backend xác minh chữ ký và các trường giao dịch. Sau đó mới ghi Payment, đánh dấu invoice, trừ điểm và cấp Membership/Enrollment/rental trong transaction. Member nhận biên lai/thông báo và xem invoice/PDF; Receptionist/Manager chỉ xem invoice theo quyền.
4. Nếu attempt thất bại, người dùng có thể retry theo checkout còn hợp lệ. Callback lặp hoặc refresh trang kết quả không phát sinh Payment/quyền lợi thứ hai.

### F03-B — Hết hạn, trả muộn và ví điểm (UC-49, UC-51–52)

1. Khi checkout gần/hết hạn, job QueryDR trước khi nhả hold. Tiền đến muộn nhưng tài nguyên còn hợp lệ thì tự cấp quyền lợi và đánh dấu `PAID_AFTER_RECONCILIATION` + `FULFILLED`.
2. Nếu đã thu tiền mà không còn cấp được chỗ/quyền, invoice ở trạng thái cần xử lý; Member chọn nhận điểm ngay (`COMPENSATED`) hoặc gửi PAYMENT Support Request, chỉ một lựa chọn có hiệu lực. Lễ tân thấy khoản Paid, not activated và có thể yêu cầu đối soát, không tự ghi nhận đã cấp.
3. Member/Receptionist nạp 10.000–10.000.000 VND bằng VNPay; sau xác minh, hệ thống cộng điểm một lần và ghi ledger. Member xem số dư, điểm đang giữ và lịch sử; nhân viên chỉ xem ví Member đã chọn trong phạm vi quyền.

### F03-C — Refund và Support Request (UC-53–60)

1. Khi trung tâm hủy lớp/rental hoặc có ngoại lệ hợp lệ, hệ thống tính cap theo invoice item, phần dịch vụ đã dùng và chuỗi chuyển lớp. Member chọn nhận điểm ngay hoặc yêu cầu hoàn chuyển khoản qua PAYMENT Support Request; hết cửa sổ lựa chọn theo quy tắc thì hệ thống tự hoàn điểm. Manager duyệt/từ chối ngoại lệ với lý do; ledger, hủy quyền lợi, audit và outbox phải nhất quán.
2. Member hoặc Receptionist tạo Support Request. PAYMENT phải chọn invoice VNPay liên quan và 1–3 ảnh bằng chứng, không nhập/lưu số tài khoản. Hạn phản hồi 48 giờ; Member hoặc Receptionist hỗ trợ gửi `Remind us` khi quá hạn theo cooldown, job nhắc người được phân công/nhân viên.
3. FrontDesk tiếp nhận/phân công/cập nhật/đóng request theo quyền; chỉ Manager xử lý PAYMENT. Manager gọi Member để lấy thông tin nhận tiền ngoài hệ thống, chuyển khoản ngoài ứng dụng, rồi ghi số tiền, ghi chú và 1–3 ảnh chứng từ. Với invoice hỗn hợp, phần điểm được hoàn theo quy tắc. Member xem và xác nhận đã nhận hoặc báo chưa nhận; không phản hồi sau 3 ngày được xử lý theo chính sách xác nhận tự động.

### F03-D — Báo cáo và thông báo (UC-61–65)

1. Manager chọn kỳ xem số Member, fill rate lớp, doanh thu theo môn/nguồn, rental và điểm phát hành/dùng/còn. Báo cáo dùng ngày thu tiền, snapshot môn và phân biệt tiền đã thu với quyền lợi đã cấp.
2. Manager chọn trường/kỳ và yêu cầu xuất PDF/CSV. Chỉ báo Completed khi file tải được; người tạo xem báo cáo của mình, Manager xem toàn bộ, có hạn lưu/xóa theo quyền.
3. Thông báo trong ứng dụng/email về thanh toán, hủy, hoàn, đổi lịch, hạn Membership được tạo ở outbox cùng giao dịch nghiệp vụ rồi gửi sau commit; lỗi email không rollback giao dịch, job retry/dedup.

## Nhánh thay thế và lỗi

| Mã | Điều kiện | Trạng thái cuối |
| --- | --- | --- |
| A1 | Giá/điểm/hold đổi hoặc checkout hết hạn | Backend từ chối amount client cũ; không thu/cấp sai, yêu cầu tạo checkout mới khi cần. |
| A2 | OTP dùng điểm hộ sai/hết hạn | Không giữ/trừ điểm; mặc định OTP 5 phút (settings 1–15), tối đa 5 lần sai, gửi lại sau 60 giây và không vượt hạn hold. |
| A3 | IPN giả, amount/TxnRef sai, return URL báo thành công nhưng chưa xác minh | Không ghi Payment hoặc cấp quyền; giữ trạng thái chờ/đối soát. |
| A4 | Callback lặp, QueryDR và IPN cùng tới | Xử lý idempotent; chỉ một Payment, ledger entry và quyền lợi. |
| A5 | Đã thu nhưng tài nguyên không còn | Không báo Fulfilled; đưa vào nhánh điểm hoặc Support Request, có audit và thông báo. |
| A6 | Member tự hủy gói/lớp/rental | Không hoàn tiền/điểm. Refund do trung tâm dựa policy và cap server tính. |
| A7 | PAYMENT thiếu invoice/ảnh, nhân viên không đủ quyền, payout chưa thực hiện | Từ chối ghi nhận; không lưu số tài khoản; không đóng request như đã thanh toán. |
| A8 | Export/email lỗi | Report Failed có thể retry; email retry qua outbox, không đảo giao dịch nghiệp vụ. |

## Hậu điều kiện và nghiệm thu

- Invoice, PaymentAttempt, Payment, PointWallet/Ledger, hold và quyền lợi khớp nhau; mọi bồi hoàn/payout có lý do, chứng từ và audit. Không có đường UI hay callback lặp cấp trùng.
- Kiểm thử: 100% điểm, điểm + VNPay, OTP hộ, callback lặp/giả, return trước IPN, thanh toán sát/sau hạn còn và hết chỗ, nạp điểm lặp, self-cancel, hoàn điểm và hoàn chuyển khoản, SLA/remind, báo cáo không cộng điểm hoàn vào tiền thu, lỗi export/email.

## Truy vết

SRS §2.3/§4.2 UC-47–65; BR-73–104, BR-110–118. Đọc thêm [API contract](../api/api-contract.md), [Source of Truth](../requirements/00-Source-of-Truth.md) và [thiết kế hệ thống mục Payment](../architecture/Center-Management-System-Design-v3.md#6-payment-vnpay-và-ví-điểm).

### Kiểm kê UC trong SRS

| Mã | Tên UC trong SRS | Actor chính |
| --- | --- | --- |
| UC-47 | Make payment | Member/Receptionist |
| UC-48 | Confirm points usage | Member/Receptionist |
| UC-49 | Reconcile VNPay payment | System; Receptionist yêu cầu đối soát |
| UC-50 | View invoices | Member/Receptionist/CenterManager |
| UC-51 | View point wallet | Member/Receptionist |
| UC-52 | Add points | Member/Receptionist |
| UC-53 | Request refund | Member/Receptionist |
| UC-54 | Review refund request | CenterManager |
| UC-55 | Choose refund option | Member |
| UC-56 | Submit Support Request | Member/Receptionist |
| UC-57 | Handle Support Request | FrontDesk/nhân viên được phân công |
| UC-58 | Remind Support Request | Member/System; Receptionist hỗ trợ |
| UC-59 | Record payout | CenterManager |
| UC-60 | Confirm payout received | Member |
| UC-61 | View management reports | CenterManager |
| UC-62 | Export report | CenterManager |
| UC-63 | Manage generated reports | Người tạo/CenterManager |
| UC-64 | View notifications | Member/Coach/staff |
| UC-65 | Send system notifications | System |
