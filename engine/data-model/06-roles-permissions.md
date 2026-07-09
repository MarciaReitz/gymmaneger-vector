# Roles and Permissions (TASK-030)

**Design only.** Any implementation or runtime change of access control is a Hard Gate.

## roles (global static catalog)

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| key | text not null unique | machine key: `admin`, `professor`, `student`, `support` |
| description | text | |
| created_at / updated_at | timestamptz | |

`key` is a code-facing identifier for a static catalog; all FKs reference `id` (see DECISIONS.md D-006).

## permissions (global static catalog)

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| key | text not null unique | e.g. `feed.post.create`, `attendance.mark`, `finance.read`, `profile.evolution.edit` |
| description | text | |
| created_at / updated_at | timestamptz | |

## role_permissions

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| role_id | uuid FK → roles | |
| permission_id | uuid FK → permissions | |

Unique `(role_id, permission_id)`. Seeded by migration (gated); never mutated at runtime in MVP.

## user_roles (per-academy assignment)

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK → academies | |
| member_id | uuid FK → academy_members | resolves to user by ID |
| role_id | uuid FK → roles | |
| status | text not null | `active` / `ended` |
| granted_by_user_id | uuid FK → user_accounts | |
| created_at / updated_at | timestamptz | |

Unique `(member_id, role_id)` where status = `active`. A user may hold different roles in different academies.

## Enforcement

- RLS policies resolve role via `academy_members` → `user_roles` by ID.
- API layer re-checks permission keys server-side before mutations.
- Every grant/revoke writes an audit event (10-audit.md).

## Baseline permission matrix (MVP)

| Permission | admin | professor | student | support |
|---|---|---|---|---|
| academy.manage | ✔ | | | |
| finance.read | ✔ | | own | ✔ |
| finance.manage | ✔ | | | |
| attendance.mark | ✔ | ✔ | | |
| attendance.checkin | | | ✔ | |
| profile.self.edit | ✔ | ✔ | ✔ | |
| profile.evolution.edit | | ✔ | | |
| feed.post.create | ✔ | ✔ | | |
| feed.read | ✔ | ✔ | ✔ | ✔ |

Final matrix requires PO validation before implementation (access-control Hard Gate).
