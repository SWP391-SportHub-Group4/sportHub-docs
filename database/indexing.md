# Indexing và quyết định tối ưu truy vấn

Nguồn: [snapshot](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/8bbe16574197a8863150dab3a1b37e6d2fbf5bc5/backend/SportHub.API/Migrations/SportHubDbContextModelSnapshot.cs) và [migration occupancy](https://github.com/SWP391-SportHub-Group4/sports-center-management-system/blob/8bbe16574197a8863150dab3a1b37e6d2fbf5bc5/backend/SportHub.API/Migrations/20260929210732_MultiSportIdentityCatalog.cs) của `main` tại `8bbe165`. Đây là **ý đồ thiết kế suy từ cột index và luồng truy vấn**, chưa phải số liệu benchmark. Danh sách đầy đủ 176 index phụ nằm trong [từ điển schema](schema/README.md); PK cũng tạo index nhưng không tính vào số đó.

| Vùng | Index/constraint hiện có | Quyết định và truy vấn được hỗ trợ |
| --- | --- | --- |
| Đăng nhập | `ix_user_accounts_email` unique trên `citext` | Tra cứu email và chặn trùng không phân biệt hoa/thường ở database. Không cần dựa vào `LOWER(email)` ở frontend. |
| Google | `ix_user_external_logins_provider_provider_user_id` và `ix_user_external_logins_user_id_provider` unique | Chặn một tài khoản provider liên kết hai user hoặc cùng user liên kết provider hai lần. |
| Lớp và catalog | `ix_classes_sport_id_status`, `ix_classes_status`, `ix_classes_code` unique | Lọc lớp theo môn/trạng thái và bảo vệ mã lớp. `ix_class_sessions_start_at_utc_status` phục vụ lịch sắp tới; `ix_class_sessions_class_id_session_no` unique giữ thứ tự buổi. |
| Giữ chỗ lớp | `ux_seat_holds_class_member_active` unique có filter `status = 0`; `ix_seat_holds_status_expires_at_utc` | Một Member không có hai hold Active cùng lớp; job quét hold đến hạn theo trạng thái và thời điểm. `ux_enrollments_class_member_confirmed` unique có filter `status = 0` chặn ghi danh Confirmed trùng. |
| Lịch phòng/Coach | `ix_room_occupancies_room_id_start_at_utc`, `ix_coach_occupancies_coach_id_start_at_utc`; hai exclusion constraints GiST | B-tree hỗ trợ tìm lịch theo chủ thể/thời điểm; GiST exclusion chặn khoảng `[start,end)` chồng nhau trên dòng `is_active`. `btree_gist` cho toán tử `=` trên ID cùng phép giao khoảng. |
| Thuê sân | `ix_court_rentals_member_id_status_start_at_utc` | Liệt kê lượt thuê của Member theo trạng thái và thời gian; `ix_court_rentals_invoice_item_id` unique có filter khi đã liên kết item. |
| Hóa đơn | `ix_invoices_invoice_number` unique; `ix_invoices_member_id_status` | Mã hóa đơn duy nhất và danh sách hóa đơn theo Member/trạng thái. Mã được sinh bằng `invoice_number_seq` để tránh va chạm khi tạo song song. |
| Checkout và VNPay | `ix_checkout_sessions_invoice_id_idempotency_key`, `ix_checkout_sessions_invoice_id_revision`, `ix_payment_attempts_vnp_txn_ref` unique | Bảo vệ retry/idempotency theo invoice và lần revision; một TxnRef không ánh xạ nhiều attempt. `ix_verified_gateway_events_provider_provider_transaction_id` unique hỗ trợ dedup sự kiện gateway. |
| Ví điểm | `ix_point_wallets_owner_user_id` unique; hai unique index `ix_point_ledger_entries_wallet_id_reference_type_reference_id_*` có filter | Một ví mỗi user; chống ghi ledger trùng theo nguồn, kể cả nhánh có/không có `invoice_item_id`. `ix_point_ledger_entries_wallet_id_created_at_utc_ledger_entry_` phục vụ lịch sử ví theo thời gian và ID. |
| Support | `ix_member_support_requests_status_sla_due_at_utc`, `ix_member_support_requests_member_id_created_at`; unique index có filter theo `invoice_id`/`invoice_item_id` khi request chưa đóng | Tìm request quá SLA, lịch sử Member và tránh hai request PAYMENT đang mở cho cùng item/invoice. |
| Thông báo | `ux_notifications_inapp_event_recipient`, `ux_notifications_email_event_recipient` unique có filter | Dedup thông báo theo sự kiện, người nhận và kênh; `ix_notifications_source_event_type_source_entity_id` tra nguồn sự kiện. |
| Audit/báo cáo | `ix_audit_logs_actor_role_timestamp`, `ix_report_exports_requested_by_user_id_created_at` | Lọc audit theo role/timestamp đã snapshot và liệt kê report của người yêu cầu theo thời gian. |

## Exclusion constraint và tính đúng khi cạnh tranh

Migration tạo các constraint vật lý sau (rút gọn đúng biểu thức):

```sql
EXCLUDE USING gist (room_id WITH =, tstzrange(start_at_utc, end_at_utc, '[)') WITH &&)
WHERE (is_active);

EXCLUDE USING gist (coach_id WITH =, tstzrange(start_at_utc, end_at_utc, '[)') WITH &&)
WHERE (is_active);
```

Tên constraint lần lượt là `ex_room_occupancies_no_overlap` và `ex_coach_occupancies_no_overlap`. Cả lớp, thuê sân và block phải ghi vào occupancy trong cùng transaction để ràng buộc DB bao phủ mọi nguồn lịch. Kiểm availability trước khi ghi chỉ hỗ trợ UX; khi hai request cạnh tranh, constraint và giao dịch quyết định. `ck_room_occupancies_range`/`ck_coach_occupancies_range` đòi `end_at_utc > start_at_utc`.

## Khi đánh giá hoặc thêm index

1. Lấy truy vấn thật và tham số lọc/sắp xếp, dùng `EXPLAIN (ANALYZE, BUFFERS)` trên dữ liệu đại diện; so với index hiện có trước khi thêm. Index làm chậm ghi và tăng dung lượng, nhất là bảng ledger/notification có nhiều insert.
2. Với index có filter, query phải chứa điều kiện tương thích (`status = 0`, `invoice_item_id IS NOT NULL`, v.v.) để planner cân nhắc index. Không đổi giá trị enum DB mà không xem lại filter SQL.
3. Kiểm các truy vấn có phân trang, composite index theo thứ tự điều kiện equality rồi thời gian; không suy index tối ưu chỉ từ tên cột. Các cột payload `jsonb` hiện không được tài liệu này hứa có GIN index; nếu cần tìm kiếm trong JSON, đo truy vấn rồi thêm migration có mục tiêu.
4. Sau migration hoặc thay đổi dữ liệu lớn, theo dõi kế hoạch thực thi và thống kê; việc có index không bảo đảm mọi truy vấn dùng nó. Không coi các index liệt kê ở đây là bằng chứng đạt SLO latency.
