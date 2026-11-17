# Schema: audit

> Sinh từ EF Core model snapshot của repo ứng dụng tại commit `8bbe16574197a8863150dab3a1b37e6d2fbf5bc5` trên `main`. Đây là mô hình EF; SQL bổ sung ngoài snapshot được ghi trong [Database Design](../README.md).

1 bảng, 10 cột. `NOT NULL`/`NULL` theo cấu hình EF; khóa và chỉ mục bên dưới dùng tên vật lý PostgreSQL.

## Mục lục

- [audit_logs](#audit-logs)

## audit_logs

Entity: `SportHub.Audit.Domain.Entities.AuditLog`. Khóa chính: `audit_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `audit_id` | `uuid` | Không | PK | — |
| `action` | `text` | Không | — | — |
| `actor_role` | `character varying(32)` | Không | — | — |
| `ip_address` | `text` | Không | — | — |
| `new_value` | `jsonb` | Có | — | — |
| `old_value` | `jsonb` | Có | — | — |
| `target_entity` | `text` | Không | — | — |
| `target_id` | `text` | Không | — | — |
| `timestamp` | `timestamp with time zone` | Không | — | — |
| `user_id` | `uuid` | Không | FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_audit_logs_user_accounts_user_id` | `user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_audit_logs_user_id` | `user_id` | Không | — |
| `ix_audit_logs_actor_role_timestamp` | `actor_role`, `timestamp` | Không | — |
