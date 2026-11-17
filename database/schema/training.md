# Schema: training

> Sinh từ EF Core model snapshot của repo ứng dụng tại commit `8bbe16574197a8863150dab3a1b37e6d2fbf5bc5` trên `main`. Đây là mô hình EF; SQL bổ sung ngoài snapshot được ghi trong [Database Design](../README.md).

3 bảng, 25 cột. `NOT NULL`/`NULL` theo cấu hình EF; khóa và chỉ mục bên dưới dùng tên vật lý PostgreSQL.

## Mục lục

- [coach_member_relationships](#coach-member-relationships)
- [workout_plan_items](#workout-plan-items)
- [workout_plans](#workout-plans)

## coach_member_relationships

Entity: `SportHub.Training.Domain.Entities.CoachMemberRelationship`. Khóa chính: `relationship_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `relationship_id` | `uuid` | Không | PK | — |
| `class_id` | `integer` | Có | FK | — |
| `coach_id` | `uuid` | Không | FK | — |
| `end_reason` | `character varying(500)` | Có | — | — |
| `ended_at` | `timestamp with time zone` | Có | — | — |
| `member_id` | `uuid` | Không | FK | — |
| `source_type` | `integer` | Không | — | — |
| `started_at` | `timestamp with time zone` | Không | — | — |
| `status` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_coach_member_relationships_classes_class_id` | `class_id` → `classes` | `Restrict` |
| `fk_coach_member_relationships_user_accounts_coach_id` | `coach_id` → `user_accounts` | `Restrict` |
| `fk_coach_member_relationships_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_coach_member_relationships_class_id` | `class_id` | Không | — |
| `ix_coach_member_relationships_member_id` | `member_id` | Không | — |
| `ix_coach_member_relationships_coach_id_member_id` | `coach_id`, `member_id` | Có | `status = 0` |


## workout_plan_items

Entity: `SportHub.Training.Domain.Entities.WorkoutPlanItem`. Khóa chính: `item_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `item_id` | `uuid` | Không | PK | — |
| `exercise` | `text` | Không | — | — |
| `notes` | `text` | Có | — | — |
| `plan_id` | `uuid` | Không | FK | — |
| `reps` | `integer` | Không | — | — |
| `sets` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_workout_plan_items_workout_plans_plan_id` | `plan_id` → `workout_plans` | `Cascade` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_workout_plan_items_plan_id` | `plan_id` | Không | — |


## workout_plans

Entity: `SportHub.Training.Domain.Entities.WorkoutPlan`. Khóa chính: `plan_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `plan_id` | `uuid` | Không | PK | — |
| `coach_id` | `uuid` | Không | FK | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `goal` | `text` | Không | — | — |
| `level` | `text` | Không | — | — |
| `member_id` | `uuid` | Không | FK | — |
| `relationship_id` | `uuid` | Không | FK | — |
| `status` | `integer` | Không | — | — |
| `updated_at` | `timestamp with time zone` | Không | — | — |
| `version` | `integer` | Không | — | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_workout_plans_user_accounts_coach_id` | `coach_id` → `user_accounts` | `Restrict` |
| `fk_workout_plans_user_accounts_member_id` | `member_id` → `user_accounts` | `Restrict` |
| `fk_workout_plans_coach_member_relationships_relationship_id` | `relationship_id` → `coach_member_relationships` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_workout_plans_coach_id` | `coach_id` | Không | — |
| `ix_workout_plans_member_id` | `member_id` | Không | — |
| `ix_workout_plans_relationship_id` | `relationship_id` | Không | — |
