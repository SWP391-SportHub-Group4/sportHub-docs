# Schema: identity

> Sinh từ EF Core model snapshot của repo ứng dụng tại commit `8bbe16574197a8863150dab3a1b37e6d2fbf5bc5` trên `main`. Đây là mô hình EF; SQL bổ sung ngoài snapshot được ghi trong [Database Design](../README.md).

16 bảng, 122 cột. `NOT NULL`/`NULL` theo cấu hình EF; khóa và chỉ mục bên dưới dùng tên vật lý PostgreSQL.

## Mục lục

- [avatar_audit_images](#avatar-audit-images)
- [avatar_deletions](#avatar-deletions)
- [avatar_report](#avatar-report)
- [coach_credentials](#coach-credentials)
- [coach_profiles](#coach-profiles)
- [email_otps](#email-otps)
- [member_support_attachments](#member-support-attachments)
- [member_support_events](#member-support-events)
- [member_support_payouts](#member-support-payouts)
- [member_support_requests](#member-support-requests)
- [roles](#roles)
- [user_accounts](#user-accounts)
- [user_credentials](#user-credentials)
- [user_external_logins](#user-external-logins)
- [user_profiles](#user-profiles)
- [user_sport_specialties](#user-sport-specialties)

## avatar_audit_images

Entity: `SportHub.Identity.Domain.Entities.AvatarAuditImage`. Khóa chính: `image_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `image_id` | `uuid` | Không | PK | — |
| `content` | `bytea` | Không | — | — |
| `content_type` | `character varying(30)` | Không | — | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `expires_at_utc` | `timestamp with time zone` | Không | — | — |
| `owner_user_id` | `uuid` | Không | — | — |
| `source_public_id` | `character varying(255)` | Có | — | — |

**Foreign keys**

Không có FK trong snapshot.

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_avatar_audit_images_expires_at_utc` | `expires_at_utc` | Không | — |
| `ix_avatar_audit_images_owner_user_id_source_public_id` | `owner_user_id`, `source_public_id` | Không | — |


## avatar_deletions

Entity: `SportHub.Identity.Domain.Entities.AvatarDeletion`. Khóa chính: `deletion_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `deletion_id` | `uuid` | Không | PK | — |
| `attempts` | `integer` | Không | — | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `last_attempt_at` | `timestamp with time zone` | Có | — | — |
| `public_id` | `character varying(255)` | Không | — | — |

**Foreign keys**

Không có FK trong snapshot.

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_avatar_deletions_public_id` | `public_id` | Có | — |


## avatar_report

Entity: `SportHub.Identity.Domain.Entities.AvatarReport`. Khóa chính: `report_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `report_id` | `uuid` | Không | PK | — |
| `avatar_url` | `character varying(2048)` | Không | — | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `reason` | `character varying(500)` | Không | — | — |
| `reporter_user_id` | `uuid` | Không | FK | — |
| `reviewed_at_utc` | `timestamp with time zone` | Có | — | — |
| `reviewed_by_user_id` | `uuid` | Có | FK | — |
| `status` | `character varying(20)` | Không | — | — |
| `target_user_id` | `uuid` | Không | FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_avatar_report_user_accounts_reporter_user_id` | `reporter_user_id` → `user_accounts` | `Restrict` |
| `fk_avatar_report_user_accounts_reviewed_by_user_id` | `reviewed_by_user_id` → `user_accounts` | `Restrict` |
| `fk_avatar_report_user_accounts_target_user_id` | `target_user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_avatar_report_reporter_user_id` | `reporter_user_id` | Không | — |
| `ix_avatar_report_reviewed_by_user_id` | `reviewed_by_user_id` | Không | — |
| `ix_avatar_report_target_user_id` | `target_user_id` | Không | — |
| `ix_avatar_report_status_created_at_utc` | `status`, `created_at_utc` | Không | — |


## coach_credentials

Entity: `SportHub.Identity.Domain.Entities.CoachCredential`. Khóa chính: `credential_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `credential_id` | `uuid` | Không | PK | — |
| `asset_deleted_at_utc` | `timestamp with time zone` | Có | — | — |
| `coach_user_id` | `uuid` | Không | FK | — |
| `content_type` | `character varying(40)` | Không | — | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `deleted_at_utc` | `timestamp with time zone` | Có | — | — |
| `file_name` | `character varying(180)` | Không | — | — |
| `file_public_id` | `character varying(255)` | Không | — | — |
| `file_url` | `character varying(1500)` | Không | — | — |
| `is_published` | `boolean` | Không | — | — |
| `issued_on` | `date` | Có | — | — |
| `issuer` | `character varying(200)` | Có | — | — |
| `kind` | `integer` | Không | — | — |
| `size_bytes` | `bigint` | Không | — | — |
| `title` | `character varying(200)` | Không | — | — |
| `updated_at_utc` | `timestamp with time zone` | Không | — | — |
| `version` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_coach_credentials_user_accounts_coach_user_id` | `coach_user_id` → `user_accounts` | `Cascade` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_coach_credentials_file_public_id` | `file_public_id` | Có | — |
| `ix_coach_credentials_coach_user_id_deleted_at_utc` | `coach_user_id`, `deleted_at_utc` | Không | — |


## coach_profiles

Entity: `SportHub.Identity.Domain.Entities.CoachProfile`. Khóa chính: `user_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `user_id` | `uuid` | Không | PK, FK | — |
| `bio` | `text` | Có | — | — |
| `philosophy` | `character varying(1000)` | Có | — | — |
| `student_count` | `integer` | Có | — | — |
| `title` | `character varying(100)` | Có | — | — |
| `years_experience` | `integer` | Có | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_coach_profiles_user_accounts_user_id` | `user_id` → `user_accounts` | `Cascade` |

**Indexes**

Không có index phụ trong snapshot; PK vẫn có index do PostgreSQL tạo.


## email_otps

Entity: `SportHub.Identity.Domain.Entities.EmailOtp`. Khóa chính: `email_otp_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `email_otp_id` | `uuid` | Không | PK | — |
| `attempts` | `integer` | Không | — | — |
| `code_hash` | `text` | Không | — | — |
| `consumed_at` | `timestamp with time zone` | Có | — | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `email` | `citext` | Không | — | — |
| `expires_at` | `timestamp with time zone` | Không | — | — |
| `purpose` | `integer` | Không | — | — |

**Foreign keys**

Không có FK trong snapshot.

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_email_otps_email_purpose` | `email`, `purpose` | Có | — |


## member_support_attachments

Entity: `SportHub.Identity.Domain.Entities.MemberSupportAttachment`. Khóa chính: `attachment_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `attachment_id` | `uuid` | Không | PK | — |
| `content` | `bytea` | Không | — | — |
| `content_type` | `character varying(50)` | Không | — | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `file_name` | `character varying(200)` | Không | — | — |
| `kind` | `character varying(20)` | Không | — | `"MEMBER_PROOF"` |
| `payout_id` | `uuid` | Có | FK | — |
| `request_id` | `uuid` | Không | FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_member_support_attachments_member_support_payouts_payout_id` | `payout_id` → `member_support_payouts` | `Restrict` |
| `fk_member_support_attachments_member_support_requests_request_` | `request_id` → `member_support_requests` | `Cascade` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_member_support_attachments_payout_id` | `payout_id` | Không | — |
| `ix_member_support_attachments_request_id` | `request_id` | Không | — |

**Check constraints**

- `ck_support_attachment_kind_payout`: `(kind = 'MEMBER_PROOF' AND payout_id IS NULL) OR (kind = 'PAYOUT_PROOF' AND payout_id IS NOT NULL)`


## member_support_events

Entity: `SportHub.Identity.Domain.Entities.MemberSupportEvent`. Khóa chính: `event_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `event_id` | `uuid` | Không | PK | — |
| `actor_user_id` | `uuid` | Có | — | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `note` | `character varying(1000)` | Có | — | — |
| `request_id` | `uuid` | Không | FK | — |
| `target_user_id` | `uuid` | Có | — | — |
| `type` | `character varying(40)` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_member_support_events_member_support_requests_request_id` | `request_id` → `member_support_requests` | `Cascade` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_member_support_events_request_id_created_at` | `request_id`, `created_at` | Không | — |


## member_support_payouts

Entity: `SportHub.Identity.Domain.Entities.MemberSupportPayout`. Khóa chính: `payout_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `payout_id` | `uuid` | Không | PK | — |
| `amount` | `numeric(18,2)` | Không | — | — |
| `invoice_id` | `uuid` | Không | FK | — |
| `invoice_item_id` | `uuid` | Có | FK | — |
| `member_confirmed_at_utc` | `timestamp with time zone` | Có | — | — |
| `member_confirmed_by` | `character varying(20)` | Có | — | — |
| `note` | `character varying(1000)` | Có | — | — |
| `points_refunded` | `integer` | Không | — | — |
| `recorded_at_utc` | `timestamp with time zone` | Không | — | — |
| `recorded_by_user_id` | `uuid` | Không | FK | — |
| `request_id` | `uuid` | Không | FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_member_support_payouts_invoices_invoice_id` | `invoice_id` → `invoices` | `Restrict` |
| `fk_member_support_payouts_invoice_items_invoice_item_id` | `invoice_item_id` → `invoice_items` | `Restrict` |
| `fk_member_support_payouts_user_accounts_recorded_by_user_id` | `recorded_by_user_id` → `user_accounts` | `Restrict` |
| `fk_member_support_payouts_member_support_requests_request_id` | `request_id` → `member_support_requests` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_member_support_payouts_invoice_id` | `invoice_id` | Không | — |
| `ix_member_support_payouts_invoice_item_id` | `invoice_item_id` | Không | — |
| `ix_member_support_payouts_recorded_by_user_id` | `recorded_by_user_id` | Không | — |
| `ix_member_support_payouts_request_id` | `request_id` | Có | — |

**Check constraints**

- `ck_member_support_payout_amount_positive`: `amount >= 0 AND points_refunded >= 0 AND (amount > 0 OR points_refunded > 0)`


## member_support_requests

Entity: `SportHub.Identity.Domain.Entities.MemberSupportRequest`. Khóa chính: `request_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `request_id` | `uuid` | Không | PK | — |
| `assigned_to_user_id` | `uuid` | Có | FK | — |
| `category` | `character varying(50)` | Không | — | — |
| `channel` | `character varying(20)` | Không | — | `"COUNTER"` |
| `closed_at` | `timestamp with time zone` | Có | — | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `description` | `character varying(2000)` | Không | — | — |
| `invoice_id` | `uuid` | Có | FK | — |
| `invoice_item_id` | `uuid` | Có | FK | — |
| `last_member_remind_at_utc` | `timestamp with time zone` | Có | — | — |
| `last_staff_remind_at_utc` | `timestamp with time zone` | Có | — | — |
| `member_id` | `uuid` | Không | FK | — |
| `progress_note` | `character varying(1000)` | Có | — | — |
| `recorded_by_user_id` | `uuid` | Không | FK | — |
| `remind_count` | `integer` | Không | — | — |
| `sla_due_at_utc` | `timestamp with time zone` | Không | — | — |
| `status` | `character varying(20)` | Không | — | — |
| `subject` | `character varying(200)` | Không | — | — |
| `updated_at` | `timestamp with time zone` | Có | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_member_support_requests_user_accounts_assigned_to_user_id` | `assigned_to_user_id` → `user_accounts` | `Restrict` |
| `fk_member_support_requests_invoices_invoice_id` | `invoice_id` → `invoices` | `Restrict` |
| `fk_member_support_requests_invoice_items_invoice_item_id` | `invoice_item_id` → `invoice_items` | `Restrict` |
| `fk_member_support_requests_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |
| `fk_member_support_requests_user_accounts_recorded_by_user_id` | `recorded_by_user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_member_support_requests_assigned_to_user_id` | `assigned_to_user_id` | Không | — |
| `ix_member_support_requests_invoice_id` | `invoice_id` | Có | `invoice_id IS NOT NULL AND invoice_item_id IS NULL AND status NOT IN ('CLOSED', 'DONE', 'REJECTED')` |
| `ix_member_support_requests_invoice_item_id` | `invoice_item_id` | Có | `invoice_item_id IS NOT NULL AND status NOT IN ('CLOSED', 'DONE', 'REJECTED')` |
| `ix_member_support_requests_recorded_by_user_id` | `recorded_by_user_id` | Không | — |
| `ix_member_support_requests_member_id_created_at` | `member_id`, `created_at` | Không | — |
| `ix_member_support_requests_status_sla_due_at_utc` | `status`, `sla_due_at_utc` | Không | — |
| `ix_member_support_requests_status_updated_at` | `status`, `updated_at` | Không | — |


## roles

Entity: `SportHub.Identity.Domain.Entities.Role`. Khóa chính: `role_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `role_id` | `integer` | Không | PK | — |
| `role_name` | `integer` | Không | — | — |

**Foreign keys**

Không có FK trong snapshot.

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_roles_role_name` | `role_name` | Có | — |


## user_accounts

Entity: `SportHub.Identity.Domain.Entities.UserAccount`. Khóa chính: `user_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `user_id` | `uuid` | Không | PK | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `email` | `citext` | Không | — | — |
| `must_change_password` | `boolean` | Không | — | — |
| `role_id` | `integer` | Không | FK | — |
| `security_stamp` | `uuid` | Không | — | — |
| `status` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_user_accounts_roles_role_id` | `role_id` → `roles` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_user_accounts_email` | `email` | Có | — |
| `ix_user_accounts_role_id` | `role_id` | Không | — |

**Check constraints**

- `ck_user_accounts_status`: `status IN (0, 2)`


## user_credentials

Entity: `SportHub.Identity.Domain.Entities.UserCredential`. Khóa chính: `user_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `user_id` | `uuid` | Không | PK, FK | — |
| `password_hash` | `text` | Có | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_user_credentials_user_accounts_user_id` | `user_id` → `user_accounts` | `Cascade` |

**Indexes**

Không có index phụ trong snapshot; PK vẫn có index do PostgreSQL tạo.


## user_external_logins

Entity: `SportHub.Identity.Domain.Entities.UserExternalLogin`. Khóa chính: `external_login_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `external_login_id` | `uuid` | Không | PK | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `provider` | `integer` | Không | — | — |
| `provider_user_id` | `text` | Không | — | — |
| `refresh_token` | `text` | Có | — | — |
| `user_id` | `uuid` | Không | FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_user_external_logins_user_accounts_user_id` | `user_id` → `user_accounts` | `Cascade` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_user_external_logins_provider_provider_user_id` | `provider`, `provider_user_id` | Có | — |
| `ix_user_external_logins_user_id_provider` | `user_id`, `provider` | Có | — |


## user_profiles

Entity: `SportHub.Identity.Domain.Entities.UserProfile`. Khóa chính: `user_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `user_id` | `uuid` | Không | PK, FK | — |
| `avatar_public_id` | `character varying(255)` | Có | — | — |
| `avatar_url` | `character varying(2048)` | Có | — | — |
| `avatar_version` | `bigint` | Không | — | — |
| `full_name` | `text` | Không | — | — |
| `phone` | `text` | Có | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_user_profiles_user_accounts_user_id` | `user_id` → `user_accounts` | `Cascade` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_user_profiles_phone` | `phone` | Có | `phone IS NOT NULL` |


## user_sport_specialties

Entity: `SportHub.Identity.Domain.Entities.UserSportSpecialty`. Khóa chính: `user_id`, `sport_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `user_id` | `uuid` | Không | PK, FK | — |
| `sport_id` | `integer` | Không | PK, FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_user_sport_specialties_sports_sport_id` | `sport_id` → `sports` | `Restrict` |
| `fk_user_sport_specialties_user_accounts_user_id` | `user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_user_sport_specialties_sport_id` | `sport_id` | Không | — |
