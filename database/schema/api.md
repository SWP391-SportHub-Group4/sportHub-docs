# Schema: api

> Sinh từ EF Core model snapshot của repo ứng dụng tại commit `8bbe16574197a8863150dab3a1b37e6d2fbf5bc5` trên `main`. Đây là mô hình EF; SQL bổ sung ngoài snapshot được ghi trong [Database Design](../README.md).

1 bảng, 4 cột. `NOT NULL`/`NULL` theo cấu hình EF; khóa và chỉ mục bên dưới dùng tên vật lý PostgreSQL.

## Mục lục

- [manager_request_reads](#manager-request-reads)

## manager_request_reads

Entity: `SportHub.API.Persistence.ManagerRequestReadReceipt`. Khóa chính: `manager_id`, `request_type`, `request_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `manager_id` | `uuid` | Không | PK, FK | — |
| `request_type` | `character varying(16)` | Có | PK | — |
| `request_id` | `uuid` | Không | PK | — |
| `read_at_utc` | `timestamp with time zone` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_manager_request_reads_user_accounts_manager_id` | `manager_id` → `user_accounts` | `Cascade` |

**Indexes**

Không có index phụ trong snapshot; PK vẫn có index do PostgreSQL tạo.
