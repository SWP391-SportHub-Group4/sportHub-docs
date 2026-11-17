# Schema: notification

> Sinh từ EF Core model snapshot của repo ứng dụng tại commit `8bbe16574197a8863150dab3a1b37e6d2fbf5bc5` trên `main`. Đây là mô hình EF; SQL bổ sung ngoài snapshot được ghi trong [Database Design](../README.md).

2 bảng, 25 cột. `NOT NULL`/`NULL` theo cấu hình EF; khóa và chỉ mục bên dưới dùng tên vật lý PostgreSQL.

## Mục lục

- [manual_notice_records](#manual-notice-records)
- [notifications](#notifications)

## manual_notice_records

Entity: `SportHub.Notification.Domain.Entities.ManualNoticeRecord`. Khóa chính: `notice_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `notice_id` | `uuid` | Không | PK | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `is_legacy` | `boolean` | Không | — | — |
| `manager_id` | `uuid` | Không | FK | — |
| `message` | `character varying(3000)` | Có | — | — |
| `recipient_count` | `integer` | Không | — | — |
| `recipients_json` | `jsonb` | Không | — | — |
| `send_email` | `boolean` | Không | — | — |
| `send_in_app` | `boolean` | Không | — | — |
| `sender_name` | `character varying(320)` | Không | — | — |
| `subject` | `character varying(150)` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_manual_notice_records_user_accounts_manager_id` | `manager_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_manual_notice_records_manager_id_created_at_utc_notice_id` | `manager_id`, `created_at_utc`, `notice_id` | Không | — |


## notifications

Entity: `SportHub.Notification.Domain.Entities.Notification`. Khóa chính: `notification_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `notification_id` | `uuid` | Không | PK | — |
| `channel` | `integer` | Không | — | — |
| `dispatch_lease_until_utc` | `timestamp with time zone` | Có | — | — |
| `last_attempt_at` | `timestamp with time zone` | Có | — | — |
| `last_error` | `character varying(120)` | Có | — | — |
| `message` | `text` | Không | — | — |
| `protected_email_payload` | `text` | Có | — | — |
| `recipient_address` | `character varying(320)` | Có | — | — |
| `retry_count` | `integer` | Không | — | — |
| `sent_at` | `timestamp with time zone` | Có | — | — |
| `source_entity_id` | `uuid` | Có | — | — |
| `source_event_type` | `integer` | Không | — | — |
| `status` | `integer` | Không | — | — |
| `user_id` | `uuid` | Có | FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_notifications_user_accounts_user_id` | `user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_notifications_user_id` | `user_id` | Không | — |
| `ix_notifications_source_event_type_source_entity_id` | `source_event_type`, `source_entity_id` | Không | — |
| `ux_notifications_email_event_recipient` | `source_event_type`, `recipient_address`, `channel`, `source_entity_id` | Có | `recipient_address IS NOT NULL AND source_entity_id IS NOT NULL AND channel = 1` |
| `ux_notifications_inapp_event_recipient` | `source_event_type`, `user_id`, `channel`, `source_entity_id` | Có | `user_id IS NOT NULL AND source_entity_id IS NOT NULL AND channel = 0` |
