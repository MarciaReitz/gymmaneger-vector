# Profile History and Evolution (TASK-035)

History is append-only; historical values are never overwritten (TASK-052 rule).

## profile_field_history (immutable)

Logs every change to history-tracked profile fields (weight, height, physical condition, health notes).

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| profile_id | uuid FK → profiles | |
| field_key | text not null | e.g. `weight_kg` |
| old_value | jsonb | |
| new_value | jsonb | |
| changed_by_user_id | uuid FK → user_accounts | author |
| created_at | timestamptz not null | |

Index: `profile_field_history(profile_id, field_key, created_at desc)`.

## student_ranks

Belt/graduation progression per modality — professor-managed (TASK-051 domain).

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK | |
| student_id | uuid FK → students | |
| modality_id | uuid FK → modalities | |
| rank_key | text not null | machine key from a per-modality rank catalog |
| awarded_by_professor_id | uuid FK → professors | |
| awarded_at | timestamptz not null | |
| created_at | timestamptz | insert-only; current rank = latest by `awarded_at` |

Index: `student_ranks(student_id, modality_id, awarded_at desc)`.

## professor_notes (immutable)

Technical/evolution comments by professors about a student.

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK | |
| student_id | uuid FK → students | |
| professor_id | uuid FK → professors | |
| note | text not null | |
| visibility | text not null | `student_visible` / `staff_only` |
| created_at | timestamptz not null | insert-only; corrections = new note |

Index: `professor_notes(student_id, created_at desc)`.

## RLS

- History readable by the student (own, and only `student_visible` notes), professors and admins of the academy.
- `profile_field_history` written by trigger/service on profile update; `student_ranks` and `professor_notes` written by professors of the academy (permission `profile.evolution.edit`).
