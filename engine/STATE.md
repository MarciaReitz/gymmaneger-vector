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
| — | Frontend integration contract v0.1 | FRONTEND_INTEGRATION_PHASE.md | 2026-07-09 |

## Gate status (per GATES.md as of 2026-07-09, Build Mode)

Auth, RBAC, Stripe, webhooks, migrations, and backend APIs may now be implemented in **dev/test mode** without stopping. Hard Gates remain only for production-affecting changes, merges to protected branches, destructive operations, and cross-agent architecture decisions.

## Blocked (dependencies, not gates)

- **`app/` scaffold** — shared area with Antigravity; creating the project skeleton is a cross-agent architecture decision → still a Hard Gate. Proposed layout in FRONTEND_INTEGRATION_PHASE.md §8 awaits Orquestra/PO decision. This blocks all backend "Implement" tasks (EPICs 05–15, 17, 18).
- **EPIC 02 accounts (Marcia)** — Supabase and Stripe dev projects are required for auth/billing implementation even in dev mode.
- **EPIC 03 (architecture, ChatGPT)** still pending; `engine/ARCHITECTURE.md` covers backend scope only and defers to TASK-019 output.
