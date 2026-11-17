# Schema: ai

> Sinh từ EF Core model snapshot của repo ứng dụng tại commit `8bbe16574197a8863150dab3a1b37e6d2fbf5bc5` trên `main`. Đây là mô hình EF; SQL bổ sung ngoài snapshot được ghi trong [Database Design](../README.md).

2 bảng, 16 cột. `NOT NULL`/`NULL` theo cấu hình EF; khóa và chỉ mục bên dưới dùng tên vật lý PostgreSQL.

## Mục lục

- [ai_logs](#ai-logs)
- [ai_system_prompts](#ai-system-prompts)

## ai_logs

Entity: `SportHub.AI.Domain.Entities.AiLog`. Khóa chính: `log_id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `log_id` | `uuid` | Không | PK | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `input_payload` | `jsonb` | Không | — | — |
| `query_type` | `text` | Không | — | — |
| `response_payload` | `jsonb` | Không | — | — |
| `response_time_ms` | `integer` | Không | — | — |
| `user_id` | `uuid` | Không | FK | — |

**Foreign keys**

| Constraint | Cột nguồn → bảng đích | On delete |
| --- | --- | --- |
| `fk_ai_logs_user_accounts_user_id` | `user_id` → `user_accounts` | `Restrict` |

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_ai_logs_user_id` | `user_id` | Không | — |


## ai_system_prompts

Entity: `SportHub.AI.Domain.Entities.SystemPrompt`. Khóa chính: `id`.

| Cột | PostgreSQL type | Null | Khóa | Mặc định trong model |
| --- | --- | --- | --- | --- |
| `id` | `character varying(64)` | Có | PK | — |
| `active_payload` | `jsonb` | Có | — | — |
| `audience` | `character varying(16)` | Không | — | — |
| `created_at` | `timestamp with time zone` | Không | — | — |
| `draft_payload` | `jsonb` | Có | — | — |
| `history_payload` | `jsonb` | Không | — | — |
| `revision` | `integer` | Không | — | — |
| `updated_at` | `timestamp with time zone` | Không | — | — |
| `updated_by` | `uuid` | Có | — | — |

**Foreign keys**

Không có FK trong snapshot.

**Indexes**

| Tên | Cột | Duy nhất | Filter / method |
| --- | --- | --- | --- |
| `ix_ai_system_prompts_audience` | `audience` | Không | — |

**Check constraints**

- `ck_ai_system_prompt_history_limit`: `jsonb_array_length(history_payload) <= 1`
- `ck_ai_system_prompt_role`: `(audience = 'Member' AND id IN ('member-rules-prompt','member-policies-prompt','member-procedures-prompt','member-general-prompt','member-api-prompt')) OR (audience = 'Coach' AND id IN ('coach-rules-prompt','coach-policies-prompt','coach-procedures-prompt','coach-general-prompt','coach-api-prompt'))`
