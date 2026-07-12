# Professors (TASK-028)

## professors

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK → academies | |
| member_id | uuid FK → academy_members, unique | resolves to user by ID |
| status | text not null | `active` / `inactive` |
| hired_at | timestamptz | |
| ended_at | timestamptz | |
| created_at / updated_at | timestamptz | |

Indexes: `professors(academy_id, status)`.

## professor_assignments

Links professors to classes (class schema: EPIC 10 / 08-attendance.md).

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK | |
| professor_id | uuid FK → professors | |
| class_id | uuid FK → classes | |
| role | text not null | `lead` / `assistant` |
| status | text not null | `active` / `ended` |
| created_at / updated_at | timestamptz | |

Unique: `(professor_id, class_id, role)` where status = `active`.

## RLS

- Academy members read; admin writes. Professors read their own assignments.
