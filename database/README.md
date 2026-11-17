# Database Design — SportHub

Tài liệu này mô tả thiết kế PostgreSQL của SportHub theo **`main` repo ứng dụng tại commit [`8bbe165`](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/tree/8bbe16574197a8863150dab3a1b37e6d2fbf5bc5)** (13/11/2026). Nguồn kỹ thuật là [EF Core model snapshot](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/8bbe16574197a8863150dab3a1b37e6d2fbf5bc5/backend/SportHub.API/Migrations/SportHubDbContextModelSnapshot.cs) cùng [migrations](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/tree/8bbe16574197a8863150dab3a1b37e6d2fbf5bc5/backend/SportHub.API/Migrations). Đây là schema **theo mã nguồn**, không phải kết quả introspect một DB đang chạy; môi trường chưa áp dụng hết migration có thể khác.

## Tài liệu trong thư mục

1. [ERD](ERD.md) — bốn sơ đồ quan hệ nghiệp vụ dễ đọc, chỉ vẽ FK thật; từ điển liệt kê mọi FK.
2. [Từ điển schema](schema/README.md) — **69 bảng, 669 cột**, kiểu PostgreSQL, nullability, PK, **138 FK**, **176 index phụ**, **54 check constraint** trong snapshot; chia theo 10 module để tra cứu.
3. [Indexing và tối ưu truy vấn](indexing.md) — các index/constraint chủ chốt, truy vấn chúng hỗ trợ và giới hạn khi đánh giá hiệu năng.

## Phạm vi và mô hình dữ liệu

API là modular monolith dùng **một PostgreSQL 16** và `SportHubDbContext`. Cấu hình EF của từng module được nạp vào cùng model, nên transaction có thể bao trùm checkout, ví điểm, giữ chỗ và cấp quyền lợi. Tên bảng/cột dùng `snake_case`; UUID cho nhiều ID nghiệp vụ, `integer` cho một số catalog, `numeric` cho tiền, `timestamp with time zone` cho thời điểm UTC, `date` cho ngày lịch, `jsonb` cho payload linh hoạt. Các giá trị `enum` trong code thường được lưu dạng `integer`; không suy ra nhãn enum từ kiểu SQL.

| Module trong snapshot | Vai trò dữ liệu | Bảng tiêu biểu |
| --- | --- | --- |
| Identity | Danh tính, role, credential, profile, hỗ trợ Member | `user_accounts`, `roles`, `user_credentials`, `member_support_requests` |
| Membership | Catalog và quyền Gym/vé ngày, hồ sơ tập | `membership_packages`, `member_packages` |
| Scheduling | Môn/phòng, lớp/buổi, ghi danh, thuê sân, occupancy, sự cố | `sports`, `rooms`, `classes`, `seat_holds`, `enrollments`, `court_rentals`, `room_occupancies` |
| Training | Quan hệ Coach–Member và workout plan | `coach_member_relationships`, `workout_plans` |
| Payment | Hóa đơn, checkout, giao dịch VNPay, ví và ledger | `invoices`, `invoice_items`, `payment_attempts`, `payments`, `point_wallets` |
| Notification | Thông báo và lịch sử gửi thủ công | `notifications`, `manual_notice_records` |
| AI | Log yêu cầu và system prompt | `ai_logs`, `ai_system_prompts` |
| Audit | Lịch sử thao tác | `audit_logs` |
| Administration / API | Thiết lập vận hành, báo cáo, landing page, dấu đọc request | `system_settings`, `report_exports`, `manager_request_reads` |

Mỗi trang [schema](schema/README.md) liệt kê bảng vật lý theo entity sở hữu. Một bảng có thể phục vụ nhiều flow; chẳng hạn `member_support_requests` nằm trong module Identity nhưng tham chiếu invoice của Payment. Không có schema PostgreSQL tách theo module trong snapshot.

## Khóa, quan hệ và ràng buộc quan trọng

- **Định danh:** `user_accounts.user_id` là gốc cho người dùng; `role_id` trỏ `roles`. `user_credentials`/`user_profiles` dùng `user_id` làm PK và FK tới tài khoản. `user_accounts.email` là `citext` có unique index, nên so sánh email không phân biệt hoa/thường ở DB.
- **Lớp:** `classes.sport_id`, `coach_id`, `default_room_id` tham chiếu danh mục/người/phòng; `class_sessions.class_id` là buổi của lớp; `enrollments.class_id` và `member_id` liên kết học viên với cả khóa. `seat_holds` giữ chỗ tạm. `ck_classes_counts` bảo vệ `0 <= confirmed_count <= reserved_count <= capacity`.
- **Lịch:** `room_occupancies` và `coach_occupancies` giữ các khoảng `[start, end)`. Migration tạo `btree_gist` và hai **exclusion constraints** `ex_room_occupancies_no_overlap`, `ex_coach_occupancies_no_overlap` trên các dòng `is_active`; hai khoảng chồng nhau cùng phòng/Coach bị từ chối, khoảng liền kề hợp lệ. Đây là SQL thủ công trong migration, không xuất hiện dưới dạng check/index EF snapshot.
- **Tài chính:** `invoices` → `invoice_items`; `checkout_sessions`/`payment_attempts` giữ lần thử; `payments` ghi khoản đã xác minh. `point_wallets.owner_user_id` unique, `point_ledger_entries` là lịch sử biến động. Check constraint giữ số dư không âm và công thức tách tiền/điểm (`cash_amount + points_applied * 1000 = total_amount` khi có checkout cycle).
- **Lịch sử và tham chiếu logic:** `source_type`/`source_id` trong occupancy, `source_event_type`/`source_entity_id` trong notification là tham chiếu theo nghiệp vụ, **không phải FK đa hình**. `avatar_audit_images.owner_user_id` cũng không có FK trong snapshot. Không nên vẽ hoặc mô tả các cột này như FK thật.

Với quan hệ xóa, xem cột `On delete` trong từ điển. Nhiều FK dùng `Restrict` để giữ hóa đơn, lịch sử và audit; một số quan hệ phụ dùng `Cascade`. Quy tắc hủy/hoàn vẫn do service và transaction quyết định, không suy từ FK cascade.

## Thứ tự thay đổi schema

1. Thêm/chỉnh entity và EF configuration ở module sở hữu; các quan hệ liên module được lắp tại API DbContext.
2. Tạo migration mới, review SQL và dữ liệu cũ cần backfill. Migration đã áp dụng không sửa lại để tránh lệch lịch sử.
3. Kiểm DB trắng và đường nâng cấp từ DB có dữ liệu, đặc biệt unique/exclusion constraints, trạng thái enum và snapshot hóa đơn.
4. Cập nhật từ điển/ERD/indexing theo migration đã merge vào `main` và ghi commit nguồn mới. Bảng trong tài liệu này không thay thế file migration khi triển khai.

Chi tiết quy tắc nghiệp vụ nằm ở [Source of Truth](../requirements/00-Source-of-Truth.md), [thiết kế hệ thống](../architecture/Center-Management-System-Design-v3.md) và [ADR-0003](../architecture/adr/0003-postgresql-ef-core.md).
