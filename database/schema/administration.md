# Schema: administration

> Sinh từ EF Core model snapshot của repo ứng dụng tại commit `8bbe16574197a8863150dab3a1b37e6d2fbf5bc5` trên `main`. Đây là mô hình EF; SQL bổ sung ngoài snapshot được ghi trong [Database Design](../README.md).

3 bảng, 23 cột. `NOT NULL`/`NULL` theo cấu hình EF; khóa và chỉ mục bên dưới dùng tên vật lý PostgreSQL.

## Mục lục

- [landing_page_content](#landing-page-content)
- [report_exports](#report-exports)
- [system_settings](#system-settings)

## landing_page_content

Entity: `SportHub.Administration.Domain.Entities.LandingPageContent`. Khóa chính: `id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `id` | `integer` | Không | PK | — |
| `revision` | `uuid` | Không | — | — |
| `updated_at` | `timestamp with time zone` | Không | — | — |
| `values_json` | `text` | Không | — | — |

**Foreign keys**

Không có FK trong snapshot.

**Indexes**

Không có index phụ trong snapshot; PK vẫn có index do PostgreSQL tạo.

**Check constraints**

- `ck_landing_page_singleton`: `id = 1`


## report_exports

Entity: `SportHub.Administration.Domain.Entities.ReportExport`. Khóa chính: `report_export_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `report_export_id` | `uuid` | Không | PK | — |
| `completed_at` | `timestamp with time zone` | Có | — | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `deleted_at` | `timestamp with time zone` | Có | — | — |
| `expires_at` | `timestamp with time zone` | Không | — | — |
| `failure_reason` | `character varying(1000)` | Có | — | — |
| `format` | `character varying(16)` | Không | — | — |
| `is_deleted` | `boolean` | Không | — | — |
| `parameters_json` | `jsonb` | Không | — | — |
| `report_type` | `character varying(64)` | Không | — | — |
| `requested_by_user_id` | `uuid` | Không | FK | — |
| `row_count` | `integer` | Không | — | — |
| `size_bytes` | `bigint` | Không | — | — |
| `status` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_report_exports_user_accounts_requested_by_user_id` | `requested_by_user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_report_exports_requested_by_user_id_created_at` | `requested_by_user_id`, `created_at` | Không | — |

**Check constraints**

- `CK_report_exports_format_allowed`: `format IN ('Csv', 'Pdf')`


## system_settings

Entity: `SportHub.Administration.Domain.Entities.SystemSetting`. Khóa chính: `key`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `key` | `character varying(64)` | Có | PK | — |
| `description` | `character varying(512)` | Không | — | — |
| `updated_at` | `timestamp with time zone` | Không | — | — |
| `updated_by_user_id` | `uuid` | Có | FK | — |
| `value` | `character varying(256)` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_system_settings_user_accounts_updated_by_user_id` | `updated_by_user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_system_settings_updated_by_user_id` | `updated_by_user_id` | Không | — |
