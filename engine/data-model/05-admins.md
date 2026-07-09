# Admins and Support (TASK-029)

## academy_admins

Administrative access records for an academy. `owner` is the accountable principal (billing contact for the SaaS subscription).

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK → academies | |
| member_id | uuid FK → academy_members, unique | resolves to user by ID |
| level | text not null | `owner` / `admin` / `support` |
| status | text not null | `active` / `ended` |
| granted_by_user_id | uuid FK → user_accounts | who granted this access |
| created_at / updated_at | timestamptz | |

Constraints:

- Unique `(member_id)` — one admin record per membership.
- Every academy must have exactly one `owner` with status `active` (enforced at API layer + partial unique index).

## Support access

`support` is read/limited-assist access (approved user type, TASK-008). Its concrete permission set is defined in 06-roles-permissions.md; granting or changing it at runtime is an access-control operation → **Hard Gate at implementation**.

## RLS

- Admins of the academy read; only `owner`/`admin` grant or revoke; all grants/revocations are audit events (10-audit.md).
