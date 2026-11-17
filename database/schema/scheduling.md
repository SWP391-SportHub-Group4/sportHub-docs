# Schema: scheduling

> Sinh từ EF Core model snapshot của repo ứng dụng tại commit `8bbe16574197a8863150dab3a1b37e6d2fbf5bc5` trên `main`. Đây là mô hình EF; SQL bổ sung ngoài snapshot được ghi trong [Database Design](../README.md).

25 bảng, 246 cột. `NOT NULL`/`NULL` theo cấu hình EF; khóa và chỉ mục bên dưới dùng tên vật lý PostgreSQL.

## Mục lục

- [attendances](#attendances)
- [class_schedule_rules](#class-schedule-rules)
- [class_session_change_requests](#class-session-change-requests)
- [class_session_reminders](#class-session-reminders)
- [class_sessions](#class-sessions)
- [class_teaching_records](#class-teaching-records)
- [class_threshold_responses](#class-threshold-responses)
- [classes](#classes)
- [coach_occupancies](#coach-occupancies)
- [court_rates](#court-rates)
- [court_rentals](#court-rentals)
- [enrollments](#enrollments)
- [gym_checkins](#gym-checkins)
- [incident_fence_steps](#incident-fence-steps)
- [incident_fences](#incident-fences)
- [incident_notices](#incident-notices)
- [room_blocks](#room-blocks)
- [room_occupancies](#room-occupancies)
- [room_opening_hours](#room-opening-hours)
- [room_types](#room-types)
- [rooms](#rooms)
- [seat_holds](#seat-holds)
- [sport_room_types](#sport-room-types)
- [sport_service_offerings](#sport-service-offerings)
- [sports](#sports)

## attendances

Entity: `SportHub.Scheduling.Domain.Entities.Attendance`. Khóa chính: `attendance_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `attendance_id` | `uuid` | Không | PK | — |
| `enrollment_id` | `uuid` | Không | FK | — |
| `last_modified_at` | `timestamp with time zone` | Có | — | — |
| `recorded_at` | `timestamp with time zone` | Không | — | — |
| `recorded_by_user_id` | `uuid` | Không | FK | — |
| `session_id` | `uuid` | Không | FK | — |
| `status` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_attendances_enrollments_enrollment_id` | `enrollment_id` → `enrollments` | `Restrict` |
| `fk_attendances_user_accounts_recorded_by_user_id` | `recorded_by_user_id` → `user_accounts` | `Restrict` |
| `fk_attendances_class_sessions_session_id` | `session_id` → `class_sessions` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_attendances_recorded_by_user_id` | `recorded_by_user_id` | Không | — |
| `ix_attendances_session_id` | `session_id` | Không | — |
| `ix_attendances_enrollment_id_session_id` | `enrollment_id`, `session_id` | Có | — |


## class_schedule_rules

Entity: `SportHub.Scheduling.Domain.Entities.ClassScheduleRule`. Khóa chính: `rule_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `rule_id` | `integer` | Không | PK | — |
| `class_id` | `integer` | Không | FK | — |
| `day_of_week` | `integer` | Không | — | — |
| `start_time_local` | `time without time zone` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_class_schedule_rules_classes_class_id` | `class_id` → `classes` | `Cascade` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_class_schedule_rules_class_id_day_of_week_start_time_local` | `class_id`, `day_of_week`, `start_time_local` | Có | — |

**Check constraints**

- `ck_class_schedule_rules_day`: `day_of_week BETWEEN 0 AND 6`


## class_session_change_requests

Entity: `SportHub.Scheduling.Domain.Entities.ClassSessionChangeRequest`. Khóa chính: `request_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `request_id` | `uuid` | Không | PK | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `original_end_at_utc` | `timestamp with time zone` | Không | — | — |
| `original_room_id` | `integer` | Không | — | — |
| `original_start_at_utc` | `timestamp with time zone` | Không | — | — |
| `outcome_snapshot_json` | `text` | Có | — | — |
| `proposed_end_at_utc` | `timestamp with time zone` | Có | — | — |
| `proposed_start_at_utc` | `timestamp with time zone` | Có | — | — |
| `reason` | `character varying(500)` | Không | — | — |
| `requested_by_user_id` | `uuid` | Không | FK | — |
| `resolution_type` | `character varying(32)` | Có | — | — |
| `result_session_id` | `uuid` | Có | FK | — |
| `review_note` | `character varying(500)` | Có | — | — |
| `reviewed_at_utc` | `timestamp with time zone` | Có | — | — |
| `reviewed_by_user_id` | `uuid` | Có | FK | — |
| `session_id` | `uuid` | Không | FK | — |
| `status` | `character varying(16)` | Không | — | — |
| `type` | `character varying(32)` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_class_session_change_requests_user_accounts_requested_by_us` | `requested_by_user_id` → `user_accounts` | `Restrict` |
| `fk_class_session_change_requests_class_sessions_result_session` | `result_session_id` → `class_sessions` | `Restrict` |
| `fk_class_session_change_requests_user_accounts_reviewed_by_use` | `reviewed_by_user_id` → `user_accounts` | `Restrict` |
| `fk_class_session_change_requests_class_sessions_session_id` | `session_id` → `class_sessions` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_class_session_change_requests_result_session_id` | `result_session_id` | Không | — |
| `ix_class_session_change_requests_reviewed_by_user_id` | `reviewed_by_user_id` | Không | — |
| `ix_class_session_change_requests_session_id` | `session_id` | Có | `status = 'PENDING'` |
| `ix_class_session_change_requests_requested_by_user_id_created_` | `requested_by_user_id`, `created_at_utc` | Không | — |
| `ix_class_session_change_requests_status_created_at_utc` | `status`, `created_at_utc` | Không | — |

**Check constraints**

- `ck_class_absence_no_proposal`: `type <> 'ABSENCE' OR (proposed_start_at_utc IS NULL AND proposed_end_at_utc IS NULL)`
- `ck_class_change_status`: `status IN ('PENDING','RESOLVED','REJECTED','WITHDRAWN')`
- `ck_class_change_type`: `type IN ('ABSENCE','SUBSTITUTE','RESCHEDULE','CANCEL_WITH_MAKEUP')`


## class_session_reminders

Entity: `SportHub.Scheduling.Domain.Entities.ClassSessionReminder`. Khóa chính: `reminder_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `reminder_id` | `uuid` | Không | PK | — |
| `hours_before` | `integer` | Không | — | — |
| `member_id` | `uuid` | Không | — | — |
| `queued_at_utc` | `timestamp with time zone` | Không | — | — |
| `session_id` | `uuid` | Không | FK | — |
| `start_at_utc` | `timestamp with time zone` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_class_session_reminders_class_sessions_session_id` | `session_id` → `class_sessions` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_class_session_reminders_session_id_member_id_start_at_utc_h` | `session_id`, `member_id`, `start_at_utc`, `hours_before` | Có | — |


## class_sessions

Entity: `SportHub.Scheduling.Domain.Entities.ClassSession`. Khóa chính: `session_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `session_id` | `uuid` | Không | PK | — |
| `class_id` | `integer` | Không | FK | — |
| `coach_id` | `uuid` | Không | FK | — |
| `end_at_utc` | `timestamp with time zone` | Không | — | — |
| `incident_report_json` | `jsonb` | Có | — | — |
| `is_makeup` | `boolean` | Không | — | — |
| `rescheduled_from_session_id` | `uuid` | Có | FK | — |
| `room_id` | `integer` | Không | FK | — |
| `session_no` | `integer` | Không | — | — |
| `start_at_utc` | `timestamp with time zone` | Không | — | — |
| `status` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_class_sessions_classes_class_id` | `class_id` → `classes` | `Restrict` |
| `fk_class_sessions_user_accounts_coach_id` | `coach_id` → `user_accounts` | `Restrict` |
| `fk_class_sessions_class_sessions_rescheduled_from_session_id` | `rescheduled_from_session_id` → `class_sessions` | `Restrict` |
| `fk_class_sessions_rooms_room_id` | `room_id` → `rooms` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_class_sessions_rescheduled_from_session_id` | `rescheduled_from_session_id` | Không | — |
| `ix_class_sessions_room_id` | `room_id` | Không | — |
| `ix_class_sessions_class_id_session_no` | `class_id`, `session_no` | Có | — |
| `ix_class_sessions_coach_id_start_at_utc` | `coach_id`, `start_at_utc` | Không | — |
| `ix_class_sessions_start_at_utc_status` | `start_at_utc`, `status` | Không | — |

**Check constraints**

- `ck_class_sessions_no`: `session_no > 0`
- `ck_class_sessions_range`: `end_at_utc > start_at_utc`


## class_teaching_records

Entity: `SportHub.Scheduling.Domain.Entities.ClassTeachingRecord`. Khóa chính: `record_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `record_id` | `uuid` | Không | PK | — |
| `class_id` | `integer` | Không | FK | — |
| `coach_id` | `uuid` | Không | FK | — |
| `content` | `character varying(8000)` | Không | — | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `due_at_utc` | `timestamp with time zone` | Có | — | — |
| `kind` | `character varying(16)` | Không | — | — |
| `member_id` | `uuid` | Có | FK | — |
| `score` | `integer` | Có | — | — |
| `session_id` | `uuid` | Có | FK | — |
| `title` | `character varying(160)` | Không | — | — |
| `updated_at_utc` | `timestamp with time zone` | Không | — | — |
| `version` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_class_teaching_records_classes_class_id` | `class_id` → `classes` | `Restrict` |
| `fk_class_teaching_records_user_accounts_coach_id` | `coach_id` → `user_accounts` | `Restrict` |
| `fk_class_teaching_records_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |
| `fk_class_teaching_records_class_sessions_session_id` | `session_id` → `class_sessions` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_class_teaching_records_coach_id` | `coach_id` | Không | — |
| `ix_class_teaching_records_member_id` | `member_id` | Không | — |
| `ix_class_teaching_records_class_id_created_at_utc` | `class_id`, `created_at_utc` | Không | — |
| `ix_class_teaching_records_session_id_member_id` | `session_id`, `member_id` | Có | `kind = 'RESULT'` |

**Check constraints**

- `ck_teaching_kind`: `kind IN ('PLAN','RESULT','NOTICE','HOMEWORK')`
- `ck_teaching_result_scope`: `kind <> 'RESULT' OR (session_id IS NOT NULL AND member_id IS NOT NULL)`
- `ck_teaching_score`: `score IS NULL OR score BETWEEN 1 AND 5`


## class_threshold_responses

Entity: `SportHub.Scheduling.Threshold.Domain.ThresholdResponse`. Khóa chính: `threshold_response_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `threshold_response_id` | `uuid` | Không | PK | — |
| `additional_invoice_id` | `uuid` | Có | FK | — |
| `choice` | `integer` | Có | — | — |
| `class_id` | `integer` | Không | FK | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `deadline_utc` | `timestamp with time zone` | Không | — | — |
| `enrollment_id` | `uuid` | Không | FK | — |
| `member_id` | `uuid` | Không | FK | — |
| `resolution_status` | `integer` | Không | — | — |
| `resolved_at_utc` | `timestamp with time zone` | Có | — | — |
| `responded_at_utc` | `timestamp with time zone` | Có | — | — |
| `target_class_id` | `integer` | Có | FK | — |
| `token_hash` | `character varying(64)` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_class_threshold_responses_invoices_additional_invoice_id` | `additional_invoice_id` → `invoices` | `Restrict` |
| `fk_class_threshold_responses_classes_class_id` | `class_id` → `classes` | `Restrict` |
| `fk_class_threshold_responses_enrollments_enrollment_id` | `enrollment_id` → `enrollments` | `Restrict` |
| `fk_class_threshold_responses_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |
| `fk_class_threshold_responses_classes_target_class_id` | `target_class_id` → `classes` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_class_threshold_responses_additional_invoice_id` | `additional_invoice_id` | Không | — |
| `ix_class_threshold_responses_enrollment_id` | `enrollment_id` | Không | — |
| `ix_class_threshold_responses_member_id` | `member_id` | Không | — |
| `ix_class_threshold_responses_target_class_id` | `target_class_id` | Không | — |
| `ix_class_threshold_responses_token_hash` | `token_hash` | Có | — |
| `ix_class_threshold_responses_class_id_enrollment_id` | `class_id`, `enrollment_id` | Có | — |
| `ix_class_threshold_responses_resolution_status_deadline_utc` | `resolution_status`, `deadline_utc` | Không | — |

**Check constraints**

- `ck_class_threshold_response_choice_target`: `(choice IS NULL AND target_class_id IS NULL) OR (choice IN (0, 2) AND target_class_id IS NULL) OR (choice = 1 AND target_class_id IS NOT NULL)`


## classes

Entity: `SportHub.Scheduling.Domain.Entities.Class`. Khóa chính: `class_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `class_id` | `integer` | Không | PK | — |
| `break_even_threshold` | `integer` | Có | — | — |
| `capacity` | `integer` | Không | — | — |
| `coach_id` | `uuid` | Có | FK | — |
| `code` | `text` | Không | — | — |
| `confirmed_count` | `integer` | Không | — | — |
| `cost_amount` | `numeric(18,0)` | Không | — | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `created_by_ai` | `boolean` | Không | — | — |
| `default_room_id` | `integer` | Không | FK | — |
| `enrollment_closed_at_utc` | `timestamp with time zone` | Có | — | — |
| `enrollment_closed_reason` | `character varying(500)` | Có | — | — |
| `enrollment_open` | `boolean` | Không | — | — |
| `name` | `text` | Không | — | — |
| `num_sessions` | `integer` | Không | — | — |
| `price` | `numeric(18,0)` | Không | — | — |
| `published_at` | `timestamp with time zone` | Có | — | — |
| `reserved_count` | `integer` | Không | — | — |
| `session_minutes` | `integer` | Có | — | — |
| `sport_id` | `integer` | Không | FK | — |
| `start_date` | `date` | Không | — | — |
| `status` | `integer` | Không | — | — |
| `threshold_deadline_utc` | `timestamp with time zone` | Có | — | — |
| `threshold_response_deadline_utc` | `timestamp with time zone` | Có | — | — |
| `threshold_status` | `integer` | Không | — | — |
| `version` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_classes_user_accounts_coach_id` | `coach_id` → `user_accounts` | `Restrict` |
| `fk_classes_rooms_default_room_id` | `default_room_id` → `rooms` | `Restrict` |
| `fk_classes_sports_sport_id` | `sport_id` → `sports` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_classes_coach_id` | `coach_id` | Không | — |
| `ix_classes_code` | `code` | Có | — |
| `ix_classes_default_room_id` | `default_room_id` | Không | — |
| `ix_classes_status` | `status` | Không | — |
| `ix_classes_sport_id_status` | `sport_id`, `status` | Không | — |

**Check constraints**

- `ck_classes_capacity`: `capacity > 0`
- `ck_classes_cost`: `cost_amount >= 0`
- `ck_classes_counts`: `confirmed_count >= 0 AND confirmed_count <= reserved_count AND reserved_count <= capacity`
- `ck_classes_num_sessions`: `num_sessions > 0`
- `ck_classes_price`: `price > 0 AND price % 1000 = 0`
- `ck_classes_session_minutes`: `session_minutes IS NULL OR session_minutes BETWEEN 1 AND 720`


## coach_occupancies

Entity: `SportHub.Scheduling.Occupancy.Domain.CoachOccupancy`. Khóa chính: `occupancy_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `occupancy_id` | `uuid` | Không | PK | — |
| `coach_id` | `uuid` | Không | FK | — |
| `end_at_utc` | `timestamp with time zone` | Không | — | — |
| `is_active` | `boolean` | Không | — | — |
| `source_id` | `uuid` | Không | — | — |
| `source_type` | `integer` | Không | — | — |
| `start_at_utc` | `timestamp with time zone` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_coach_occupancies_user_accounts_coach_id` | `coach_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_coach_occupancies_coach_id_start_at_utc` | `coach_id`, `start_at_utc` | Không | — |
| `ix_coach_occupancies_source_type_source_id` | `source_type`, `source_id` | Có | — |

**Check constraints**

- `ck_coach_occupancies_range`: `end_at_utc > start_at_utc`


## court_rates

Entity: `SportHub.Scheduling.Catalog.Domain.CourtRate`. Khóa chính: `rate_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `rate_id` | `integer` | Không | PK | — |
| `days_of_week` | `text` | Không | — | — |
| `end_time_local` | `time without time zone` | Không | — | — |
| `is_active` | `boolean` | Không | — | — |
| `price_per_hour` | `numeric(18,0)` | Không | — | — |
| `room_id` | `integer` | Có | FK | — |
| `room_type_id` | `integer` | Không | FK | — |
| `sport_id` | `integer` | Có | FK | — |
| `start_time_local` | `time without time zone` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_court_rates_rooms_room_id` | `room_id` → `rooms` | `Cascade` |
| `fk_court_rates_room_types_room_type_id` | `room_type_id` → `room_types` | `Restrict` |
| `fk_court_rates_sports_sport_id` | `sport_id` → `sports` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_court_rates_room_id` | `room_id` | Không | — |
| `ix_court_rates_sport_id` | `sport_id` | Không | — |
| `ix_court_rates_room_type_id_is_active` | `room_type_id`, `is_active` | Không | — |

**Check constraints**

- `ck_court_rates_price`: `price_per_hour > 0 AND price_per_hour % 1000 = 0`
- `ck_court_rates_range`: `end_time_local > start_time_local`


## court_rentals

Entity: `SportHub.Scheduling.Rental.Domain.CourtRental`. Khóa chính: `court_rental_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `court_rental_id` | `uuid` | Không | PK | — |
| `cancel_reason` | `character varying(500)` | Có | — | — |
| `cancellation_incident_id` | `uuid` | Có | FK | — |
| `cancelled_at_utc` | `timestamp with time zone` | Có | — | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `end_at_utc` | `timestamp with time zone` | Không | — | — |
| `invoice_id` | `uuid` | Có | FK | — |
| `invoice_item_id` | `uuid` | Có | FK | — |
| `member_id` | `uuid` | Không | FK | — |
| `price_snapshot_json` | `jsonb` | Không | — | — |
| `room_id` | `integer` | Không | FK | — |
| `sport_id` | `integer` | Không | FK | — |
| `start_at_utc` | `timestamp with time zone` | Không | — | — |
| `status` | `integer` | Không | — | — |
| `total_price` | `numeric(12,2)` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_court_rentals_incident_notices_cancellation_incident_id` | `cancellation_incident_id` → `incident_notices` | `Restrict` |
| `fk_court_rentals_invoices_invoice_id` | `invoice_id` → `invoices` | `Restrict` |
| `fk_court_rentals_invoice_items_invoice_item_id` | `invoice_item_id` → `invoice_items` | `Restrict` |
| `fk_court_rentals_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |
| `fk_court_rentals_rooms_room_id` | `room_id` → `rooms` | `Restrict` |
| `fk_court_rentals_sports_sport_id` | `sport_id` → `sports` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_court_rentals_cancellation_incident_id` | `cancellation_incident_id` | Không | — |
| `ix_court_rentals_invoice_id` | `invoice_id` | Không | — |
| `ix_court_rentals_invoice_item_id` | `invoice_item_id` | Có | `invoice_item_id IS NOT NULL` |
| `ix_court_rentals_room_id` | `room_id` | Không | — |
| `ix_court_rentals_sport_id` | `sport_id` | Không | — |
| `ix_court_rentals_member_id_status_start_at_utc` | `member_id`, `status`, `start_at_utc` | Không | — |

**Check constraints**

- `ck_court_rentals_range`: `end_at_utc > start_at_utc`


## enrollments

Entity: `SportHub.Scheduling.Domain.Entities.Enrollment`. Khóa chính: `enrollment_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `enrollment_id` | `uuid` | Không | PK | — |
| `center_cancellation_paid_balance_vnd` | `numeric(18,2)` | Có | — | — |
| `center_cancellation_refund_points` | `integer` | Có | — | — |
| `class_id` | `integer` | Không | FK | — |
| `ended_at` | `timestamp with time zone` | Có | — | — |
| `enrolled_at` | `timestamp with time zone` | Không | — | — |
| `invoice_item_id` | `uuid` | Có | FK | — |
| `member_id` | `uuid` | Không | FK | — |
| `source_enrollment_id` | `uuid` | Có | FK | — |
| `status` | `integer` | Không | — | — |
| `transfer_difference_invoice_item_id` | `uuid` | Có | FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_enrollments_classes_class_id` | `class_id` → `classes` | `Restrict` |
| `fk_enrollments_invoice_items_invoice_item_id` | `invoice_item_id` → `invoice_items` | `Restrict` |
| `fk_enrollments_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |
| `fk_enrollments_enrollments_source_enrollment_id` | `source_enrollment_id` → `enrollments` | `Restrict` |
| `fk_enrollments_invoice_items_transfer_difference_invoice_item_` | `transfer_difference_invoice_item_id` → `invoice_items` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ux_enrollments_invoice_item` | `invoice_item_id` | Có | `invoice_item_id IS NOT NULL AND status = 0` |
| `ix_enrollments_source_enrollment_id` | `source_enrollment_id` | Không | — |
| `ix_enrollments_transfer_difference_invoice_item_id` | `transfer_difference_invoice_item_id` | Không | — |
| `ux_enrollments_class_member_confirmed` | `class_id`, `member_id` | Có | `status = 0` |
| `ix_enrollments_member_id_status` | `member_id`, `status` | Không | — |


## gym_checkins

Entity: `SportHub.Scheduling.Domain.Entities.GymCheckIn`. Khóa chính: `check_in_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `check_in_id` | `uuid` | Không | PK | — |
| `check_in_time` | `timestamp with time zone` | Không | — | — |
| `checked_in_by_user_id` | `uuid` | Không | FK | — |
| `member_id` | `uuid` | Không | FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_gym_checkins_user_accounts_checked_in_by_user_id` | `checked_in_by_user_id` → `user_accounts` | `Restrict` |
| `fk_gym_checkins_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_gym_checkins_checked_in_by_user_id` | `checked_in_by_user_id` | Không | — |
| `ix_gym_checkins_member_id_check_in_time` | `member_id`, `check_in_time` | Không | — |


## incident_fence_steps

Entity: `SportHub.Scheduling.Rental.Domain.IncidentFenceStep`. Khóa chính: `step_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `step_id` | `uuid` | Không | PK | — |
| `actor_user_id` | `uuid` | Không | — | — |
| `after_coach_id` | `uuid` | Không | — | — |
| `after_room_id` | `integer` | Không | — | — |
| `after_start_utc` | `timestamp with time zone` | Không | — | — |
| `before_coach_id` | `uuid` | Không | — | — |
| `before_end_utc` | `timestamp with time zone` | Không | — | — |
| `before_room_id` | `integer` | Không | — | — |
| `before_start_utc` | `timestamp with time zone` | Không | — | — |
| `block_created_by_user_id` | `uuid` | Có | — | — |
| `block_reason` | `character varying(1000)` | Có | — | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `fence_id` | `uuid` | Không | FK | — |
| `kind` | `integer` | Không | — | — |
| `makeup_session_id` | `uuid` | Có | — | — |
| `room_block_id` | `uuid` | Có | — | — |
| `session_id` | `uuid` | Có | — | — |
| `state` | `integer` | Không | — | — |
| `undo_note` | `character varying(300)` | Có | — | — |
| `undone_at_utc` | `timestamp with time zone` | Có | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_incident_fence_steps_incident_fences_fence_id` | `fence_id` → `incident_fences` | `Cascade` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_incident_fence_steps_session_id` | `session_id` | Không | — |
| `ix_incident_fence_steps_fence_id_created_at_utc` | `fence_id`, `created_at_utc` | Không | — |


## incident_fences

Entity: `SportHub.Scheduling.Rental.Domain.IncidentFence`. Khóa chính: `fence_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `fence_id` | `uuid` | Không | PK | — |
| `closed_at_utc` | `timestamp with time zone` | Có | — | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `created_by_user_id` | `uuid` | Không | — | — |
| `end_at_utc` | `timestamp with time zone` | Không | — | — |
| `expires_at_utc` | `timestamp with time zone` | Không | — | — |
| `reason` | `character varying(500)` | Không | — | — |
| `resolved_incident_id` | `uuid` | Có | — | — |
| `room_id` | `integer` | Có | — | — |
| `scope` | `integer` | Không | — | — |
| `start_at_utc` | `timestamp with time zone` | Không | — | — |
| `status` | `integer` | Không | — | — |

**Foreign keys**

Không có FK trong snapshot.

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_incident_fences_status_start_at_utc_end_at_utc` | `status`, `start_at_utc`, `end_at_utc` | Không | — |

**Check constraints**

- `ck_incident_fence_range`: `end_at_utc > start_at_utc`


## incident_notices

Entity: `SportHub.Scheduling.Rental.Domain.IncidentNotice`. Khóa chính: `incident_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `incident_id` | `uuid` | Không | PK | — |
| `closes_gym` | `boolean` | Không | — | — |
| `created_at_utc` | `timestamp with time zone` | Không | — | — |
| `created_by_user_id` | `uuid` | Không | FK | — |
| `day_pass_members_json` | `jsonb` | Không | — | — |
| `cancelled_at_utc` | `timestamp with time zone` | Có | — | — |
| `cancelled_by_user_id` | `uuid` | Có | — | — |
| `day_pass_notice_sent_at_utc` | `timestamp with time zone` | Có | — | — |
| `end_at_utc` | `timestamp with time zone` | Không | — | — |
| `extended_member_count` | `integer` | Không | — | — |
| `membership_extension_days` | `integer` | Không | — | — |
| `reason` | `character varying(500)` | Không | — | — |
| `refunds_json` | `jsonb` | Không | — | — |
| `request_fingerprint` | `character varying(64)` | Có | — | — |
| `resolution_summary` | `character varying(2000)` | Không | — | — |
| `room_id` | `integer` | Có | FK | — |
| `scope` | `integer` | Không | — | — |
| `start_at_utc` | `timestamp with time zone` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_incident_notices_user_accounts_created_by_user_id` | `created_by_user_id` → `user_accounts` | `Restrict` |
| `fk_incident_notices_rooms_room_id` | `room_id` → `rooms` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_incident_notices_created_by_user_id` | `created_by_user_id` | Không | — |
| `ix_incident_notices_room_id` | `room_id` | Không | — |
| `ix_incident_notices_scope_room_id_start_at_utc` | `scope`, `room_id`, `start_at_utc` | Không | — |

**Check constraints**

- `ck_incident_membership_extension`: `membership_extension_days BETWEEN 0 AND 31 AND extended_member_count >= 0`
- `ck_incident_notice_range`: `end_at_utc > start_at_utc`


## room_blocks

Entity: `SportHub.Scheduling.Catalog.Domain.RoomBlock`. Khóa chính: `block_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `block_id` | `uuid` | Không | PK | — |
| `created_by_user_id` | `uuid` | Không | — | — |
| `end_at_utc` | `timestamp with time zone` | Không | — | — |
| `incident_id` | `uuid` | Có | FK | — |
| `reason` | `text` | Không | — | — |
| `room_id` | `integer` | Không | FK | — |
| `start_at_utc` | `timestamp with time zone` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_room_blocks_incident_notices_incident_id` | `incident_id` → `incident_notices` | `Restrict` |
| `fk_room_blocks_rooms_room_id` | `room_id` → `rooms` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_room_blocks_incident_id` | `incident_id` | Không | — |
| `ix_room_blocks_room_id_start_at_utc` | `room_id`, `start_at_utc` | Không | — |

**Check constraints**

- `ck_room_blocks_range`: `end_at_utc > start_at_utc`


## room_occupancies

Entity: `SportHub.Scheduling.Occupancy.Domain.RoomOccupancy`. Khóa chính: `occupancy_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `occupancy_id` | `uuid` | Không | PK | — |
| `end_at_utc` | `timestamp with time zone` | Không | — | — |
| `is_active` | `boolean` | Không | — | — |
| `room_id` | `integer` | Không | FK | — |
| `source_id` | `uuid` | Không | — | — |
| `source_type` | `integer` | Không | — | — |
| `start_at_utc` | `timestamp with time zone` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_room_occupancies_rooms_room_id` | `room_id` → `rooms` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_room_occupancies_room_id_start_at_utc` | `room_id`, `start_at_utc` | Không | — |
| `ix_room_occupancies_source_type_source_id` | `source_type`, `source_id` | Có | — |

**Check constraints**

- `ck_room_occupancies_range`: `end_at_utc > start_at_utc`


## room_opening_hours

Entity: `SportHub.Scheduling.Catalog.Domain.RoomOpeningHour`. Khóa chính: `room_id`, `day_of_week`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `room_id` | `integer` | Không | PK, FK | — |
| `day_of_week` | `integer` | Không | PK | — |
| `close_time_local` | `time without time zone` | Không | — | — |
| `open_time_local` | `time without time zone` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_room_opening_hours_rooms_room_id` | `room_id` → `rooms` | `Restrict` |

**Indexes**

Không có index phụ trong snapshot; PK vẫn có index do PostgreSQL tạo.

**Check constraints**

- `ck_room_opening_hours_day`: `day_of_week BETWEEN 0 AND 6`
- `ck_room_opening_hours_range`: `close_time_local > open_time_local`


## room_types

Entity: `SportHub.Scheduling.Catalog.Domain.RoomType`. Khóa chính: `room_type_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `room_type_id` | `integer` | Không | PK | — |
| `is_active` | `boolean` | Không | — | `true` |
| `name` | `citext` | Không | — | — |

**Foreign keys**

Không có FK trong snapshot.

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_room_types_name` | `name` | Có | — |


## rooms

Entity: `SportHub.Scheduling.Domain.Entities.Room`. Khóa chính: `room_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `room_id` | `integer` | Không | PK | — |
| `allow_rental` | `boolean` | Không | — | `false` |
| `deleted_at_utc` | `timestamp with time zone` | Có | — | — |
| `capacity` | `integer` | Không | — | — |
| `is_active` | `boolean` | Không | — | — |
| `name` | `text` | Không | — | — |
| `rental_booking_opens_at_utc` | `timestamp with time zone` | Có | — | — |
| `room_type_id` | `integer` | Có | FK | — |
| `sport_id` | `integer` | Có | FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_rooms_room_types_room_type_id` | `room_type_id` → `room_types` | `Restrict` |
| `fk_rooms_sports_sport_id` | `sport_id` → `sports` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_rooms_name` | `name` | Có | `deleted_at_utc IS NULL` |
| `ix_rooms_room_type_id` | `room_type_id` | Không | — |
| `ix_rooms_sport_id` | `sport_id` | Không | — |


## seat_holds

Entity: `SportHub.Scheduling.Domain.Entities.SeatHold`. Khóa chính: `hold_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `hold_id` | `uuid` | Không | PK | — |
| `class_id` | `integer` | Không | FK | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `expires_at_utc` | `timestamp with time zone` | Không | — | — |
| `invoice_id` | `uuid` | Có | FK | — |
| `member_id` | `uuid` | Không | FK | — |
| `status` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_seat_holds_classes_class_id` | `class_id` → `classes` | `Restrict` |
| `fk_seat_holds_invoices_invoice_id` | `invoice_id` → `invoices` | `Restrict` |
| `fk_seat_holds_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_seat_holds_invoice_id` | `invoice_id` | Không | — |
| `ix_seat_holds_member_id` | `member_id` | Không | — |
| `ux_seat_holds_class_member_active` | `class_id`, `member_id` | Có | `status = 0` |
| `ix_seat_holds_status_expires_at_utc` | `status`, `expires_at_utc` | Không | — |


## sport_room_types

Entity: `SportHub.Scheduling.Catalog.Domain.SportRoomType`. Khóa chính: `sport_id`, `room_type_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `sport_id` | `integer` | Không | PK, FK | — |
| `room_type_id` | `integer` | Không | PK, FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_sport_room_types_room_types_room_type_id` | `room_type_id` → `room_types` | `Restrict` |
| `fk_sport_room_types_sports_sport_id` | `sport_id` → `sports` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_sport_room_types_room_type_id` | `room_type_id` | Không | — |


## sport_service_offerings

Entity: `SportHub.Scheduling.Catalog.Domain.SportServiceOffering`. Khóa chính: `offering_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `offering_id` | `integer` | Không | PK | — |
| `default_max_capacity` | `integer` | Có | — | — |
| `default_session_minutes` | `integer` | Có | — | — |
| `is_enabled` | `boolean` | Không | — | — |
| `service_type` | `integer` | Không | — | — |
| `sport_id` | `integer` | Không | FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_sport_service_offerings_sports_sport_id` | `sport_id` → `sports` | `Cascade` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_sport_service_offerings_sport_id_service_type` | `sport_id`, `service_type` | Có | — |

**Check constraints**

- `ck_sport_service_offerings_defaults`: `(service_type = 1 AND default_session_minutes > 0 AND default_max_capacity > 0) OR (service_type <> 1 AND default_session_minutes IS NULL AND default_max_capacity IS NULL)`
- `ck_sport_service_offerings_type`: `service_type BETWEEN 0 AND 3`


## sports

Entity: `SportHub.Scheduling.Catalog.Domain.Sport`. Khóa chính: `sport_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `sport_id` | `integer` | Không | PK | — |
| `code` | `citext` | Không | — | — |
| `description` | `text` | Có | — | — |
| `image_url` | `text` | Có | — | — |
| `is_active` | `boolean` | Không | — | — |
| `name` | `citext` | Không | — | — |
| `sort_order` | `integer` | Không | — | — |

**Foreign keys**

Không có FK trong snapshot.

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_sports_code` | `code` | Có | — |
| `ix_sports_name` | `name` | Có | — |
