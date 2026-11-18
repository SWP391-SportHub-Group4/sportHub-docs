# API Testing

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

[Mục lục tài liệu kiểm thử](README.md)
