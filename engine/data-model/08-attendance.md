# Classes, Attendance, QR Check-in (TASK-032)

Class/session hierarchy anticipates EPIC 10 (TASK-077) so attendance FKs are stable.

## classes

A recurring class definition.

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK → academies | |
| branch_id | uuid FK → branches | |
| modality_id | uuid FK → modalities | |
| room_id | uuid FK → rooms | |
| name | text not null | display only |
| schedule_rrule | text | recurrence rule (RFC 5545 subset) |
| capacity_override | integer | defaults to room capacity |
| status | text not null | `active` / `archived` |
| created_at / updated_at | timestamptz | |

## class_sessions

A concrete occurrence of a class.

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK | |
| class_id | uuid FK → classes | |
| room_id | uuid FK → rooms | may differ from class default |
| starts_at / ends_at | timestamptz not null | |
| status | text not null | `scheduled` / `in_progress` / `done` / `canceled` |
| created_at / updated_at | timestamptz | |

Indexes: `class_sessions(academy_id, starts_at)`, `class_sessions(class_id)`.

## attendance_records

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK | |
| session_id | uuid FK → class_sessions | |
| student_id | uuid FK → students | |
| status | text not null | `present` / `late` / `absent` |
| method | text not null | `qr` / `manual` |
| checked_in_at | timestamptz | |
| marked_by_user_id | uuid FK → user_accounts, nullable | null = QR self check-in |
| created_at / updated_at | timestamptz | |

Unique: `(session_id, student_id)`.

## qr_tokens (TASK-059/060 design)

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK | |
| session_id | uuid FK → class_sessions | |
| token_hash | text not null unique | only the hash is stored |
| expires_at | timestamptz not null | short-lived |
| revoked_at | timestamptz | |
| created_at | timestamptz | |

Validation flow (backend-only): scan → hash lookup by ID → check expiry/revocation → verify student's active membership and status → upsert `attendance_records` with `method = 'qr'`. Occupancy (TASK-061) = count of `present` per session vs capacity, computed from `attendance_records`.

## RLS

- Professors/admins of the academy manage sessions and records; students insert their own QR check-in only via the validated endpoint; students read their own records.
