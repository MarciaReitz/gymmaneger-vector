# Data Model — Overview (EPIC 04)

Owner: Claude. Design documents only — SQL migrations will be written at implementation time and **migration apply is a Hard Gate**.

All tables follow `engine/STANDARDS.md`: uuid PKs, `created_at`/`updated_at`, `academy_id` on tenant tables, IDs-only relationships, no hard deletes, history over overwrite.

## Domain map

| File | Task | Domain |
|---|---|---|
| 01-academy.md | TASK-025 | Academies, branches, rooms |
| 02-users.md | TASK-026 + TASK-049 | Users, profiles, academy membership |
| 03-students.md | TASK-027 | Student records and status |
| 04-professors.md | TASK-028 | Professor records and assignments |
| 05-admins.md | TASK-029 | Admin and support access structures |
| 06-roles-permissions.md | TASK-030 | Roles, permissions, mappings |
| 07-billing.md | TASK-031 | Plans, subscriptions, invoices, payments |
| 08-attendance.md | TASK-032 | Classes, sessions, presence, QR |
| 09-feed.md | TASK-033 | Posts, categories, targeting, reactions |
| 10-audit.md | TASK-034 | Audit trail and event logs |
| 11-profile-history.md | TASK-035 | Profile evolution and professor notes |

## Cross-domain invariants

- Tenant isolation: every tenant table has `academy_id`; RLS validates membership via `academy_members`.
- A user's relationship to an academy always flows through `academy_members` (one row per user per academy).
- Paid access state is written only by the Stripe webhook handler.
- Immutable tables: `payments`, `audit_events`, `webhook_events`, `profile_field_history`, `professor_notes`.
