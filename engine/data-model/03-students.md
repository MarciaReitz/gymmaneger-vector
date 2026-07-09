# Students (TASK-027)

## students

A student is a role-specific record bound to an academy membership. A user can be a student in more than one academy (one row each).

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK → academies | |
| member_id | uuid FK → academy_members, unique with kind | resolves to user by ID |
| status | text not null | `active` / `inactive` / `overdue` / `frozen` — current snapshot |
| enrolled_at | timestamptz not null | |
| left_at | timestamptz | |
| created_at / updated_at | timestamptz | |

Unique: `(member_id)` — one student record per membership.
Indexes: `students(academy_id, status)`.

## student_status_history

Status is computed by the status engine (EPIC 12) from finance, attendance, and profile data; every transition is recorded, never overwritten.

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK | |
| student_id | uuid FK → students | |
| from_status / to_status | text not null | |
| reason_code | text not null | machine code (e.g. `invoice_overdue`, `manual_freeze`) |
| changed_by_user_id | uuid FK → user_accounts, nullable | null = system |
| created_at | timestamptz | insert-only |

## RLS

- Students read their own record; professors and admins of the academy read all; writes: admin (and system jobs) only.
