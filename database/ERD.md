# ERD — quan hệ dữ liệu SportHub

Nguồn: EF Core model snapshot và migrations ở commit [`8bbe165`](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/tree/8bbe16574197a8863150dab3a1b37e6d2fbf5bc5) của `main` repo ứng dụng. Các sơ đồ chia theo luồng để đọc được trên GitHub; chúng chỉ thể hiện **FK thật** của những bảng nghiệp vụ chính. Toàn bộ 69 bảng và 138 FK nằm trong [từ điển schema](schema/README.md). Ký hiệu `||` là một, `o{` là nhiều, `o|` là không hoặc một; cột nullable và hành vi xóa xem từ điển.

## Danh tính và Membership

```mermaid
erDiagram
    roles ||--o{ user_accounts : role_id
    user_accounts ||--o| user_credentials : user_id
    user_accounts ||--o| user_profiles : user_id
    user_accounts ||--o{ user_external_logins : user_id
    user_accounts ||--o{ member_packages : member_id
    membership_packages ||--o{ member_packages : package_id
    user_accounts ||--o{ gym_checkins : member_id
    user_accounts ||--o{ coach_member_relationships : coach_id
    user_accounts ||--o{ coach_member_relationships : member_id
```

`user_accounts` là bảng định danh chung cho năm role; credential, profile và đăng nhập ngoài tách riêng. `member_packages` là quyền Gym/vé đã cấp, còn `membership_packages` là catalog. Sự hiện diện của FK không tự chứng minh role của user: các dịch vụ nghiệp vụ vẫn kiểm role/trạng thái.

## Danh mục, lớp, chỗ giữ và sân

```mermaid
erDiagram
    sports ||--o{ classes : sport_id
    rooms ||--o{ classes : default_room_id
    user_accounts ||--o{ classes : coach_id
    classes ||--o{ class_sessions : class_id
    rooms ||--o{ class_sessions : room_id
    user_accounts ||--o{ class_sessions : coach_id
    classes ||--o{ seat_holds : class_id
    user_accounts ||--o{ seat_holds : member_id
    classes ||--o{ enrollments : class_id
    user_accounts ||--o{ enrollments : member_id
    enrollments ||--o{ attendances : enrollment_id
    class_sessions ||--o{ attendances : session_id
    rooms ||--o{ court_rentals : room_id
    user_accounts ||--o{ court_rentals : member_id
    rooms ||--o{ room_occupancies : room_id
    user_accounts ||--o{ coach_occupancies : coach_id
```

`room_occupancies` và `coach_occupancies` dùng cặp `source_type`/`source_id` để chỉ nguồn chiếm lịch; cặp này không phải FK đa hình đến lớp/rental/block. Việc chống trùng thực tế do exclusion constraints trên khoảng thời gian và cùng các transaction ghi occupancy thực hiện. `SeatHold` là chỗ tạm; `Enrollment` là ghi danh cả khóa sau thanh toán.

## Hóa đơn, thanh toán và ví điểm

```mermaid
erDiagram
    user_accounts ||--o{ invoices : member_id
    invoices ||--o{ invoice_items : invoice_id
    invoices ||--o{ checkout_sessions : invoice_id
    invoices ||--o{ payment_attempts : invoice_id
    checkout_sessions ||--o{ payment_attempts : checkout_session_id
    invoices ||--o{ payments : invoice_id
    payment_attempts ||--o{ payments : payment_attempt_id
    user_accounts ||--o| point_wallets : owner_user_id
    point_wallets ||--o{ point_ledger_entries : wallet_id
    invoice_items ||--o{ point_ledger_entries : invoice_item_id
    invoice_items ||--o{ payment_adjustments : invoice_item_id
    payments ||--o{ payment_adjustments : payment_id
    invoice_items ||--o| member_packages : invoice_item_id
    invoice_items ||--o| court_rentals : invoice_item_id
    invoice_items ||--o| enrollments : invoice_item_id
```

`invoice_items` giữ snapshot món/giá; `checkout_sessions` và `payment_attempts` giữ vòng đời thử thanh toán. `payments` chỉ ghi khoản đã xác minh. Các cạnh từ invoice item đến quyền lợi có thể nullable/được ràng buộc bằng unique có điều kiện; xem [schema Payment](schema/payment.md) và [schema Scheduling](schema/scheduling.md). Không suy rằng một invoice item đồng thời cấp cả ba loại quyền lợi.

## Hỗ trợ, tập luyện và sự kiện

```mermaid
erDiagram
    user_accounts ||--o{ member_support_requests : member_id
    invoices ||--o{ member_support_requests : invoice_id
    invoice_items ||--o{ member_support_requests : invoice_item_id
    member_support_requests ||--o| member_support_payouts : request_id
    member_support_requests ||--o{ member_support_events : request_id
    member_support_requests ||--o{ member_support_attachments : request_id
    user_accounts ||--o{ workout_plans : coach_id
    user_accounts ||--o{ workout_plans : member_id
    workout_plans ||--o{ workout_plan_items : plan_id
    user_accounts ||--o{ notifications : user_id
    user_accounts ||--o{ audit_logs : user_id
    user_accounts ||--o{ ai_logs : user_id
```

Một số bản ghi như log, audit hoặc media có tham chiếu logic thay vì FK để giữ lịch sử khi tài nguyên nguồn đổi. Từ điển schema phân biệt rõ FK thật với cột ID thông thường.
