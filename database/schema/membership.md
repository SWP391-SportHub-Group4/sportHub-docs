# Schema: membership

> Sinh từ EF Core model snapshot của repo ứng dụng tại commit `8bbe16574197a8863150dab3a1b37e6d2fbf5bc5` trên `main`. Đây là mô hình EF; SQL bổ sung ngoài snapshot được ghi trong [Database Design](../README.md).

4 bảng, 44 cột. `NOT NULL`/`NULL` theo cấu hình EF; khóa và chỉ mục bên dưới dùng tên vật lý PostgreSQL.

## Mục lục

- [member_body_measurements](#member-body-measurements)
- [member_packages](#member-packages)
- [member_training_profiles](#member-training-profiles)
- [membership_packages](#membership-packages)

## member_body_measurements

Entity: `SportHub.Membership.Domain.Entities.MemberBodyMeasurement`. Khóa chính: `measurement_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `measurement_id` | `uuid` | Không | PK | — |
| `height_cm` | `numeric(5,1)` | Không | — | — |
| `measured_at` | `timestamp with time zone` | Không | — | — |
| `member_id` | `uuid` | Không | FK | — |
| `weight_kg` | `numeric(5,1)` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_member_body_measurements_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_member_body_measurements_member_id_measured_at` | `member_id`, `measured_at` | Không | — |

**Check constraints**

- `ck_member_body_measurement_range`: `height_cm BETWEEN 100 AND 250 AND weight_kg BETWEEN 30 AND 300`


## member_packages

Entity: `SportHub.Membership.Domain.Entities.MemberPackage`. Khóa chính: `member_package_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `member_package_id` | `uuid` | Không | PK | — |
| `duration_days_snapshot` | `integer` | Có | — | — |
| `end_date` | `date` | Không | — | — |
| `expires_at_utc` | `timestamp with time zone` | Có | — | — |
| `invoice_item_id` | `uuid` | Có | FK | — |
| `member_id` | `uuid` | Không | FK | — |
| `package_id` | `integer` | Không | FK | — |
| `remaining_sessions` | `integer` | Có | — | — |
| `stacking_approval_reason` | `text` | Có | — | — |
| `stacking_approved_by_user_id` | `uuid` | Có | FK | — |
| `start_date` | `date` | Không | — | — |
| `status` | `integer` | Không | — | — |
| `version` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_member_packages_invoice_items_invoice_item_id` | `invoice_item_id` → `invoice_items` | `Restrict` |
| `fk_member_packages_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |
| `fk_member_packages_membership_packages_package_id` | `package_id` → `membership_packages` | `Restrict` |
| `fk_member_packages_user_accounts_stacking_approved_by_user_id` | `stacking_approved_by_user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_member_packages_invoice_item_id` | `invoice_item_id` | Có | `invoice_item_id IS NOT NULL` |
| `ix_member_packages_package_id` | `package_id` | Không | — |
| `ix_member_packages_stacking_approved_by_user_id` | `stacking_approved_by_user_id` | Không | — |
| `ix_member_packages_member_id_status` | `member_id`, `status` | Không | — |

**Check constraints**

- `CK_member_packages_remaining_sessions_non_negative`: `remaining_sessions IS NULL OR remaining_sessions >= 0`
- `CK_member_packages_stacking_approval_complete`: `(stacking_approved_by_user_id IS NULL AND stacking_approval_reason IS NULL) OR (stacking_approved_by_user_id IS NOT NULL AND stacking_approval_reason IS NOT NULL)`


## member_training_profiles

Entity: `SportHub.Membership.Domain.Entities.MemberTrainingProfile`. Khóa chính: `profile_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `profile_id` | `uuid` | Không | PK | — |
| `activity_level` | `integer` | Có | — | — |
| `body_build` | `character varying(20)` | Có | — | — |
| `date_of_birth` | `date` | Có | — | — |
| `dietary_preference` | `integer` | Không | — | — |
| `experience_level` | `integer` | Không | — | — |
| `food_allergies` | `character varying(500)` | Có | — | — |
| `goal` | `text` | Không | — | — |
| `goal_type` | `integer` | Có | — | — |
| `height_cm` | `numeric(5,1)` | Có | — | — |
| `member_id` | `uuid` | Không | FK | — |
| `notes` | `text` | Có | — | — |
| `session_minutes` | `integer` | Có | — | — |
| `sex` | `integer` | Có | — | — |
| `target_weight_kg` | `numeric(5,1)` | Có | — | — |
| `training_days_per_week` | `integer` | Có | — | — |
| `updated_at` | `timestamp with time zone` | Không | — | — |
| `weight_kg` | `numeric(5,1)` | Có | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_member_training_profiles_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_member_training_profiles_member_id` | `member_id` | Có | — |


## membership_packages

Entity: `SportHub.Membership.Domain.Entities.MembershipPackage`. Khóa chính: `package_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `package_id` | `integer` | Không | PK | — |
| `description` | `text` | Có | — | — |
| `duration_days` | `integer` | Không | — | — |
| `is_active` | `boolean` | Không | — | — |
| `name` | `text` | Không | — | — |
| `original_price` | `numeric(18,0)` | Có | — | — |
| `price` | `numeric(18,0)` | Không | — | — |
| `session_limit` | `integer` | Có | — | — |

**Foreign keys**

Không có FK trong snapshot.

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_membership_packages_name` | `name` | Có | — |
