# Schema: payment

> Sinh từ EF Core model snapshot của repo ứng dụng tại commit `8bbe16574197a8863150dab3a1b37e6d2fbf5bc5` trên `main`. Đây là mô hình EF; SQL bổ sung ngoài snapshot được ghi trong [Database Design](../README.md).

12 bảng, 154 cột. `NOT NULL`/`NULL` theo cấu hình EF; khóa và chỉ mục bên dưới dùng tên vật lý PostgreSQL.

## Mục lục

- [checkout_sessions](#checkout-sessions)
- [invoice_items](#invoice-items)
- [invoices](#invoices)
- [payment_adjustments](#payment-adjustments)
- [payment_attempts](#payment-attempts)
- [payments](#payments)
- [point_confirmations](#point-confirmations)
- [point_credit_requests](#point-credit-requests)
- [point_gifts](#point-gifts)
- [point_ledger_entries](#point-ledger-entries)
- [point_wallets](#point-wallets)
- [verified_gateway_events](#verified-gateway-events)

## checkout_sessions

Entity: `SportHub.Payment.Domain.Entities.CheckoutSession`. Khóa chính: `checkout_session_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `checkout_session_id` | `uuid` | Không | PK | — |
| `class_id` | `integer` | Có | — | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `expires_at_utc` | `timestamp with time zone` | Không | — | — |
| `idempotency_key` | `character varying(120)` | Không | — | — |
| `invoice_id` | `uuid` | Không | FK | — |
| `kind` | `character varying(30)` | Không | — | — |
| `resource_hold_id` | `uuid` | Có | — | — |
| `revision` | `integer` | Không | — | — |
| `state` | `character varying(30)` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_checkout_sessions_invoices_invoice_id` | `invoice_id` → `invoices` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_checkout_sessions_invoice_id_idempotency_key` | `invoice_id`, `idempotency_key` | Có | — |
| `ix_checkout_sessions_invoice_id_revision` | `invoice_id`, `revision` | Có | — |
| `ix_checkout_sessions_invoice_id_state` | `invoice_id`, `state` | Không | — |


## invoice_items

Entity: `SportHub.Payment.Domain.Entities.InvoiceItem`. Khóa chính: `item_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `item_id` | `uuid` | Không | PK | — |
| `class_id` | `integer` | Có | FK | — |
| `court_rental_id` | `uuid` | Có | FK | — |
| `description` | `character varying(500)` | Không | — | — |
| `invoice_id` | `uuid` | Không | FK | — |
| `item_type` | `integer` | Không | — | — |
| `line_amount` | `numeric(18,0)` | Không | — | — |
| `member_package_id` | `uuid` | Có | FK | — |
| `quantity` | `integer` | Không | — | — |
| `related_entity_id` | `uuid` | Có | — | — |
| `source_invoice_item_id` | `uuid` | Có | FK | — |
| `sport_id` | `integer` | Có | FK | — |
| `sport_name_snapshot` | `character varying(200)` | Có | — | — |
| `unit_price` | `numeric(18,0)` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_invoice_items_classes_class_id` | `class_id` → `classes` | `Restrict` |
| `fk_invoice_items_court_rentals_court_rental_id` | `court_rental_id` → `court_rentals` | `Restrict` |
| `fk_invoice_items_invoices_invoice_id` | `invoice_id` → `invoices` | `Restrict` |
| `fk_invoice_items_member_packages_member_package_id` | `member_package_id` → `member_packages` | `Restrict` |
| `fk_invoice_items_invoice_items_source_invoice_item_id` | `source_invoice_item_id` → `invoice_items` | `Restrict` |
| `fk_invoice_items_sports_sport_id` | `sport_id` → `sports` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_invoice_items_class_id` | `class_id` | Không | — |
| `ix_invoice_items_court_rental_id` | `court_rental_id` | Không | — |
| `ix_invoice_items_invoice_id` | `invoice_id` | Không | — |
| `ix_invoice_items_member_package_id` | `member_package_id` | Không | — |
| `ix_invoice_items_source_invoice_item_id` | `source_invoice_item_id` | Không | — |
| `ix_invoice_items_sport_id` | `sport_id` | Không | — |

**Check constraints**

- `CK_invoice_items_line_amount_matches`: `line_amount = unit_price * quantity`
- `CK_invoice_items_quantity_positive`: `quantity > 0`
- `CK_invoice_items_unit_price_non_negative`: `unit_price >= 0`


## invoices

Entity: `SportHub.Payment.Domain.Entities.Invoice`. Khóa chính: `invoice_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `invoice_id` | `uuid` | Không | PK | — |
| `cash_amount` | `numeric(18,0)` | Không | — | — |
| `checkout_cycle_id` | `uuid` | Có | — | — |
| `checkout_revision` | `integer` | Không | — | — |
| `hold_expires_at_utc` | `timestamp with time zone` | Có | — | — |
| `invoice_number` | `text` | Không | — | — |
| `issued_at` | `timestamp with time zone` | Không | — | — |
| `issued_by_user_id` | `uuid` | Không | FK | — |
| `member_id` | `uuid` | Không | FK | — |
| `member_package_id` | `uuid` | Có | FK | — |
| `paid_at_utc` | `timestamp with time zone` | Có | — | — |
| `paid_via` | `text` | Có | — | — |
| `points_applied` | `integer` | Không | — | — |
| `reconciliation_required` | `boolean` | Không | — | — |
| `resolution_idempotency_key` | `character varying(120)` | Có | — | — |
| `status` | `integer` | Không | — | — |
| `total_amount` | `numeric(18,0)` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_invoices_user_accounts_issued_by_user_id` | `issued_by_user_id` → `user_accounts` | `Restrict` |
| `fk_invoices_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |
| `fk_invoices_member_packages_member_package_id` | `member_package_id` → `member_packages` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_invoices_invoice_number` | `invoice_number` | Có | — |
| `ix_invoices_issued_by_user_id` | `issued_by_user_id` | Không | — |
| `ix_invoices_member_package_id` | `member_package_id` | Không | — |
| `ix_invoices_member_id_status` | `member_id`, `status` | Không | — |

**Check constraints**

- `ck_invoice_point_split`: `points_applied >= 0 AND cash_amount >= 0 AND (checkout_cycle_id IS NULL OR cash_amount + points_applied::numeric * 1000 = total_amount)`


## payment_adjustments

Entity: `SportHub.Payment.Domain.Entities.PaymentAdjustment`. Khóa chính: `adjustment_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `adjustment_id` | `uuid` | Không | PK | — |
| `amount` | `numeric(18,0)` | Không | — | — |
| `approved_at_utc` | `timestamp with time zone` | Có | — | — |
| `approved_by_user_id` | `uuid` | Có | FK | — |
| `approved_points` | `integer` | Không | — | — |
| `center_fault` | `boolean` | Không | — | — |
| `completed_at_utc` | `timestamp with time zone` | Có | — | — |
| `completed_by_user_id` | `uuid` | Có | FK | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `invoice_id` | `uuid` | Không | FK | — |
| `invoice_item_id` | `uuid` | Có | FK | — |
| `legacy_payout_unverified` | `boolean` | Không | — | — |
| `payment_id` | `uuid` | Có | FK | — |
| `point_ledger_entry_id` | `uuid` | Có | FK | — |
| `reason` | `text` | Không | — | — |
| `refund_method` | `integer` | Có | — | — |
| `refund_reference_code` | `character varying(100)` | Có | — | — |
| `requested_amount` | `numeric(18,0)` | Không | — | — |
| `requested_by_user_id` | `uuid` | Không | FK | — |
| `resolved_at` | `timestamp with time zone` | Có | — | — |
| `status` | `integer` | Không | — | — |
| `system_calculated_points` | `integer` | Không | — | — |
| `type` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_payment_adjustments_user_accounts_approved_by_user_id` | `approved_by_user_id` → `user_accounts` | `Restrict` |
| `fk_payment_adjustments_user_accounts_completed_by_user_id` | `completed_by_user_id` → `user_accounts` | `Restrict` |
| `fk_payment_adjustments_invoices_invoice_id` | `invoice_id` → `invoices` | `Restrict` |
| `fk_payment_adjustments_invoice_items_invoice_item_id` | `invoice_item_id` → `invoice_items` | `Restrict` |
| `fk_payment_adjustments_payments_payment_id` | `payment_id` → `payments` | `Restrict` |
| `fk_payment_adjustments_point_ledger_entries_point_ledger_entry` | `point_ledger_entry_id` → `point_ledger_entries` | `Restrict` |
| `fk_payment_adjustments_user_accounts_requested_by_user_id` | `requested_by_user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_payment_adjustments_approved_by_user_id` | `approved_by_user_id` | Không | — |
| `ix_payment_adjustments_completed_by_user_id` | `completed_by_user_id` | Không | — |
| `ix_payment_adjustments_invoice_id` | `invoice_id` | Không | — |
| `ix_payment_adjustments_invoice_item_id` | `invoice_item_id` | Không | — |
| `ix_payment_adjustments_payment_id` | `payment_id` | Không | — |
| `ix_payment_adjustments_point_ledger_entry_id` | `point_ledger_entry_id` | Không | — |
| `ix_payment_adjustments_requested_by_user_id` | `requested_by_user_id` | Không | — |

**Check constraints**

- `CK_payment_adjustments_completed_has_date`: `status <> 3 OR legacy_payout_unverified OR completed_at_utc IS NOT NULL`
- `CK_payment_adjustments_refund_completed_evidence`: `(type <> 0 OR status <> 3)\nOR legacy_payout_unverified\nOR (invoice_item_id IS NOT NULL AND point_ledger_entry_id IS NOT NULL\n    AND approved_points > 0 AND completed_at_utc IS NOT NULL)\nOR (completed_at_utc IS NOT NULL\n    AND completed_by_user_id IS NOT NULL\n    AND refund_method IS NOT NULL)`


## payment_attempts

Entity: `SportHub.Payment.Domain.Entities.PaymentAttempt`. Khóa chính: `payment_attempt_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `payment_attempt_id` | `uuid` | Không | PK | — |
| `amount` | `numeric(18,0)` | Không | — | — |
| `cash_snapshot` | `numeric(18,0)` | Không | — | — |
| `checkout_session_id` | `uuid` | Có | FK | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `invoice_id` | `uuid` | Không | FK | — |
| `points_snapshot` | `integer` | Không | — | — |
| `provider_transaction_id` | `character varying(100)` | Có | — | — |
| `status` | `integer` | Không | — | — |
| `verified_at_utc` | `timestamp with time zone` | Có | — | — |
| `verified_result` | `character varying(40)` | Có | — | — |
| `vnp_expire_date` | `timestamp with time zone` | Không | — | — |
| `vnp_txn_ref` | `character varying(100)` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_payment_attempts_checkout_sessions_checkout_session_id` | `checkout_session_id` → `checkout_sessions` | `Restrict` |
| `fk_payment_attempts_invoices_invoice_id` | `invoice_id` → `invoices` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_payment_attempts_checkout_session_id` | `checkout_session_id` | Không | — |
| `ix_payment_attempts_vnp_txn_ref` | `vnp_txn_ref` | Có | — |
| `ix_payment_attempts_invoice_id_status` | `invoice_id`, `status` | Không | — |


## payments

Entity: `SportHub.Payment.Domain.Entities.Payment`. Khóa chính: `payment_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `payment_id` | `uuid` | Không | PK | — |
| `amount` | `numeric(18,0)` | Không | — | — |
| `invoice_id` | `uuid` | Không | FK | — |
| `method` | `integer` | Không | — | — |
| `paid_at` | `timestamp with time zone` | Không | — | — |
| `payment_attempt_id` | `uuid` | Có | FK | — |
| `received_by_user_id` | `uuid` | Không | FK | — |
| `reference_code` | `text` | Có | — | — |
| `status` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_payments_invoices_invoice_id` | `invoice_id` → `invoices` | `Restrict` |
| `fk_payments_payment_attempts_payment_attempt_id` | `payment_attempt_id` → `payment_attempts` | `Restrict` |
| `fk_payments_user_accounts_received_by_user_id` | `received_by_user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_payments_invoice_id` | `invoice_id` | Không | — |
| `ix_payments_payment_attempt_id` | `payment_attempt_id` | Không | — |
| `ix_payments_received_by_user_id` | `received_by_user_id` | Không | — |


## point_confirmations

Entity: `SportHub.Payment.Wallet.Domain.PointConfirmation`. Khóa chính: `point_confirmation_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `point_confirmation_id` | `uuid` | Không | PK | — |
| `checkout_cycle_id` | `uuid` | Không | — | — |
| `checkout_revision` | `integer` | Không | — | — |
| `code_hash` | `character varying(64)` | Không | — | — |
| `code_salt` | `character varying(32)` | Không | — | — |
| `consumed_at_utc` | `timestamp with time zone` | Có | — | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `expires_at_utc` | `timestamp with time zone` | Không | — | — |
| `failed_attempts` | `integer` | Không | — | — |
| `invoice_id` | `uuid` | Không | FK | — |
| `member_id` | `uuid` | Không | FK | — |
| `points` | `integer` | Không | — | — |
| `requested_by_user_id` | `uuid` | Không | FK | — |
| `revoked_at_utc` | `timestamp with time zone` | Có | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_point_confirmations_invoices_invoice_id` | `invoice_id` → `invoices` | `Restrict` |
| `fk_point_confirmations_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |
| `fk_point_confirmations_user_accounts_requested_by_user_id` | `requested_by_user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_point_confirmations_requested_by_user_id` | `requested_by_user_id` | Không | — |
| `ix_point_confirmations_invoice_id_created_at_utc` | `invoice_id`, `created_at_utc` | Không | — |
| `ix_point_confirmations_member_id_created_at_utc` | `member_id`, `created_at_utc` | Không | — |

**Check constraints**

- `ck_point_confirmation_attempts`: `failed_attempts >= 0 AND failed_attempts <= 5 AND points > 0`


## point_credit_requests

Entity: `SportHub.Payment.Wallet.Domain.PointCreditRequest`. Khóa chính: `request_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `request_id` | `uuid` | Không | PK | — |
| `decided_at_utc` | `timestamp with time zone` | Có | — | — |
| `decided_by_user_id` | `uuid` | Có | FK | — |
| `decision_reason` | `character varying(500)` | Có | — | — |
| `ledger_entry_id` | `uuid` | Có | FK | — |
| `owner_user_id` | `uuid` | Không | FK | — |
| `points` | `integer` | Không | — | — |
| `reason` | `character varying(1000)` | Không | — | — |
| `requested_at_utc` | `timestamp with time zone` | Không | — | — |
| `requested_by_user_id` | `uuid` | Không | FK | — |
| `status` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_point_credit_requests_user_accounts_decided_by_user_id` | `decided_by_user_id` → `user_accounts` | `Restrict` |
| `fk_point_credit_requests_point_ledger_entries_ledger_entry_id` | `ledger_entry_id` → `point_ledger_entries` | `Restrict` |
| `fk_point_credit_requests_user_accounts_owner_user_id` | `owner_user_id` → `user_accounts` | `Restrict` |
| `fk_point_credit_requests_user_accounts_requested_by_user_id` | `requested_by_user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_point_credit_requests_decided_by_user_id` | `decided_by_user_id` | Không | — |
| `ix_point_credit_requests_ledger_entry_id` | `ledger_entry_id` | Có | — |
| `ix_point_credit_requests_requested_by_user_id` | `requested_by_user_id` | Không | — |
| `ix_point_credit_requests_owner_user_id_requested_at_utc` | `owner_user_id`, `requested_at_utc` | Không | — |
| `ix_point_credit_requests_status_requested_at_utc` | `status`, `requested_at_utc` | Không | — |

**Check constraints**

- `ck_point_credit_positive`: `points > 0`
- `ck_point_credit_resolution`: `(status = 0 AND decided_by_user_id IS NULL AND decided_at_utc IS NULL AND ledger_entry_id IS NULL AND decision_reason IS NULL) OR\r\n(status = 1 AND decided_by_user_id IS NOT NULL AND decided_at_utc IS NOT NULL AND ledger_entry_id IS NOT NULL AND decision_reason IS NULL) OR\r\n(status = 2 AND decided_by_user_id IS NOT NULL AND decided_at_utc IS NOT NULL AND ledger_entry_id IS NULL AND decision_reason IS NOT NULL)`
- `ck_point_credit_separate_approver`: `decided_by_user_id IS NULL OR decided_by_user_id <> requested_by_user_id`
- `ck_point_credit_status`: `status IN (0, 1, 2)`


## point_gifts

Entity: `SportHub.Payment.Wallet.Domain.PointGift`. Khóa chính: `gift_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `gift_id` | `uuid` | Không | PK | — |
| `batch_id` | `uuid` | Không | — | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `incident_id` | `uuid` | Có | FK | — |
| `ledger_entry_id` | `uuid` | Không | FK | — |
| `manager_id` | `uuid` | Không | FK | — |
| `member_agreed` | `boolean` | Không | — | — |
| `member_id` | `uuid` | Không | FK | — |
| `points` | `integer` | Không | — | — |
| `reason` | `character varying(1000)` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_point_gifts_incident_notices_incident_id` | `incident_id` → `incident_notices` | `Restrict` |
| `fk_point_gifts_point_ledger_entries_ledger_entry_id` | `ledger_entry_id` → `point_ledger_entries` | `Restrict` |
| `fk_point_gifts_user_accounts_manager_id` | `manager_id` → `user_accounts` | `Restrict` |
| `fk_point_gifts_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_point_gifts_created_at_utc` | `created_at_utc` | Không | — |
| `ix_point_gifts_ledger_entry_id` | `ledger_entry_id` | Có | — |
| `ix_point_gifts_manager_id` | `manager_id` | Không | — |
| `ix_point_gifts_member_id` | `member_id` | Không | — |
| `ix_point_gifts_batch_id_member_id` | `batch_id`, `member_id` | Có | — |
| `ix_point_gifts_incident_id_member_id` | `incident_id`, `member_id` | Có | `incident_id IS NOT NULL` |

**Check constraints**

- `ck_point_gift_positive`: `points > 0`


## point_ledger_entries

Entity: `SportHub.Payment.Wallet.Domain.PointLedgerEntry`. Khóa chính: `ledger_entry_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `ledger_entry_id` | `uuid` | Không | PK | — |
| `actor_user_id` | `uuid` | Có | FK | — |
| `available_after` | `integer` | Không | — | — |
| `available_delta` | `integer` | Không | — | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `entry_type` | `integer` | Không | — | — |
| `held_after` | `integer` | Không | — | — |
| `held_delta` | `integer` | Không | — | — |
| `invoice_item_id` | `uuid` | Có | FK | — |
| `note` | `character varying(1000)` | Có | — | — |
| `points` | `integer` | Không | — | — |
| `reference_id` | `uuid` | Không | — | — |
| `reference_type` | `character varying(80)` | Không | — | — |
| `wallet_id` | `uuid` | Không | FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_point_ledger_entries_user_accounts_actor_user_id` | `actor_user_id` → `user_accounts` | `Restrict` |
| `fk_point_ledger_entries_invoice_items_invoice_item_id` | `invoice_item_id` → `invoice_items` | `Restrict` |
| `fk_point_ledger_entries_point_wallets_wallet_id` | `wallet_id` → `point_wallets` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_point_ledger_entries_actor_user_id` | `actor_user_id` | Không | — |
| `ix_point_ledger_entries_invoice_item_id` | `invoice_item_id` | Không | — |
| `ix_point_ledger_entries_wallet_id_created_at_utc_ledger_entry_` | `wallet_id`, `created_at_utc`, `ledger_entry_id` | Không | — |
| `ix_point_ledger_entries_wallet_id_reference_type_reference_id_` | `wallet_id`, `reference_type`, `reference_id`, `entry_type` | Có | `invoice_item_id IS NULL` |
| `ix_point_ledger_entries_wallet_id_reference_type_reference_id_1` | `wallet_id`, `reference_type`, `reference_id`, `entry_type`, `invoice_item_id` | Có | `invoice_item_id IS NOT NULL` |

**Check constraints**

- `ck_point_ledger_balance`: `available_after >= 0 AND held_after >= 0 AND points > 0`
- `ck_point_ledger_deltas`: `(entry_type = 0 AND available_delta = -points AND held_delta = points) OR\n(entry_type = 1 AND available_delta = points AND held_delta = -points) OR\n(entry_type = 2 AND available_delta = 0 AND held_delta = -points) OR\n(entry_type = 3 AND available_delta = points AND held_delta = 0) OR\n(entry_type = 4 AND held_delta = 0 AND (available_delta = points OR available_delta = -points))`


## point_wallets

Entity: `SportHub.Payment.Wallet.Domain.PointWallet`. Khóa chính: `wallet_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `wallet_id` | `uuid` | Không | PK | — |
| `available_points` | `integer` | Không | — | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `held_points` | `integer` | Không | — | — |
| `owner_user_id` | `uuid` | Không | FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_point_wallets_user_accounts_owner_user_id` | `owner_user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_point_wallets_owner_user_id` | `owner_user_id` | Có | — |

**Check constraints**

- `ck_point_wallets_balance`: `available_points >= 0 AND held_points >= 0`


## verified_gateway_events

Entity: `SportHub.Payment.Domain.Entities.VerifiedGatewayEvent`. Khóa chính: `verified_gateway_event_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `verified_gateway_event_id` | `uuid` | Không | PK | — |
| `amount` | `numeric(18,0)` | Không | — | — |
| `last_error` | `character varying(1000)` | Có | — | — |
| `payment_attempt_id` | `uuid` | Không | FK | — |
| `processed_at_utc` | `timestamp with time zone` | Có | — | — |
| `processing_status` | `character varying(40)` | Không | — | — |
| `provider` | `character varying(30)` | Không | — | — |
| `provider_paid_at_utc` | `timestamp with time zone` | Không | — | — |
| `provider_transaction_id` | `character varying(100)` | Không | — | — |
| `response_code` | `character varying(10)` | Không | — | — |
| `retry_count` | `integer` | Không | — | — |
| `transaction_reference` | `character varying(100)` | Không | — | — |
| `transaction_status` | `character varying(10)` | Không | — | — |
| `verified_at_utc` | `timestamp with time zone` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_verified_gateway_events_payment_attempts_payment_attempt_id` | `payment_attempt_id` → `payment_attempts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_verified_gateway_events_payment_attempt_id` | `payment_attempt_id` | Không | — |
| `ix_verified_gateway_events_processing_status_verified_at_utc` | `processing_status`, `verified_at_utc` | Không | — |
| `ix_verified_gateway_events_provider_provider_transaction_id` | `provider`, `provider_transaction_id` | Có | — |
