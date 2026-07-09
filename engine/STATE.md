# Backend State Ledger

Owner: Claude. Task completion state for Claude-owned tasks. `roadmap/` is ChatGPT-owned; ChatGPT should sync ROADMAP_MASTER from this ledger (DECISIONS.md D-005).

## Completed

| Task | Title | Deliverable | Date |
|---|---|---|---|
| TASK-025 | Design academy tables | data-model/01-academy.md | 2026-07-09 |
| TASK-026 | Design user tables | data-model/02-users.md | 2026-07-09 |
| TASK-027 | Design student tables | data-model/03-students.md | 2026-07-09 |
| TASK-028 | Design professor tables | data-model/04-professors.md | 2026-07-09 |
| TASK-029 | Design admin tables | data-model/05-admins.md | 2026-07-09 |
| TASK-030 | Design role and permission tables | data-model/06-roles-permissions.md | 2026-07-09 |
| TASK-031 | Design plan and subscription tables | data-model/07-billing.md | 2026-07-09 |
| TASK-032 | Design attendance tables | data-model/08-attendance.md | 2026-07-09 |
| TASK-033 | Design feed tables | data-model/09-feed.md | 2026-07-09 |
| TASK-034 | Design audit and logs tables | data-model/10-audit.md | 2026-07-09 |
| TASK-035 | Design profile history tables | data-model/11-profile-history.md | 2026-07-09 |
| TASK-049 | Define profile schema | data-model/02-users.md (profiles) | 2026-07-09 |
| TASK-121 | Environment variables plan | ENVIRONMENT.md | 2026-07-09 |

## Awaiting PO approval (Hard Gates)

| Task | Reason |
|---|---|
| TASK-041 | Stripe product structure drafted in INTEGRATIONS.md — anything affecting Stripe is a Hard Gate |
| TASK-030 matrix | Permission matrix drafted — access-control implementation is a Hard Gate |
| EPIC 05 (TASK-036..040) | Auth implementation — Hard Gate + requires Supabase project (TASK-011, Marcia) |
| EPIC 06 (TASK-042..048) | Billing implementation — Hard Gate + requires Stripe account (TASK-012, Marcia) |

## Blocked (dependencies)

- All remaining Claude "Implement" tasks (EPICs 07–15, 17, 18) require: app scaffold decision (shared `app/` area — needs Orquestra coordination with Antigravity), TASK-129 approval (start of implementation), and EPIC 02 manual accounts (Marcia).
- EPIC 03 (architecture, ChatGPT) still pending; `engine/ARCHITECTURE.md` covers backend scope only and defers to TASK-019 output.
