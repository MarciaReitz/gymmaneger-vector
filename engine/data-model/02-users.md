# Users, Profiles, Membership (TASK-026 + TASK-049)

## user_accounts

Mirrors Supabase `auth.users` (same `id`). App-level account state lives here; credentials stay in Supabase Auth.

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | = `auth.users.id` |
| status | text not null | `active` / `suspended` / `deleted` (soft) |
| created_at / updated_at | timestamptz | |

## profiles (TASK-049 — lightweight fields only)

One profile per user, global (not per academy). The profile is the single evolving record (TASK-010). Every change is logged in `profile_field_history` (11-profile-history.md); current values live here.

| Column | Type | Edited by | Notes |
|---|---|---|---|
| id | uuid PK | | |
| user_id | uuid FK → user_accounts, unique | | |
| full_name | text not null | student | display only, never a key |
| photo_url | text | student | Supabase Storage |
| birth_date | date | student | |
| weight_kg | numeric(5,2) | student | history-tracked |
| height_cm | numeric(5,1) | student | history-tracked |
| physical_condition | text | student | history-tracked |
| health_notes | text | student | injuries, restrictions; history-tracked |
| emergency_contact_name | text | student | |
| emergency_contact_phone | text | student | |
| created_at / updated_at | timestamptz | | |

Professor-managed evolution data (belt, technical notes) is **not** stored on the profile row — it lives in `student_ranks` and `professor_notes` (11-profile-history.md), keeping the profile lightweight and permissions clean (field-level rules: TASK-053, gated at implementation).

## academy_members

The single junction between users and academies. All tenant access checks resolve through this table by ID.

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK → academies | |
| user_id | uuid FK → user_accounts | |
| status | text not null | `active` / `suspended` / `ended` |
| joined_at | timestamptz not null | |
| ended_at | timestamptz | |
| created_at / updated_at | timestamptz | |

Unique: `(academy_id, user_id)`. Indexes: `academy_members(user_id)`, `academy_members(academy_id, status)`.

## RLS

- `profiles`: owner reads/writes own student-editable fields; professors/admins of academies where the user is a member read; field-level write rules enforced at API layer (implementation gated).
- `academy_members`: members read own rows; academy `admin` manages rows of their academy.
