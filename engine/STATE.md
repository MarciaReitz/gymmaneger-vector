# Backend State Ledger

Owner: Claude. Task completion state for Claude-owned tasks. `roadmap/` is ChatGPT-owned; ChatGPT should sync ROADMAP_MASTER from this ledger (DECISIONS.md D-005).

Code: `gymmanager/backend/` + `gymmanager/shared/translation.ts`, branch `claude/backend-bootstrap` (23 unit tests passing, typecheck clean). Design docs: `engine/`.

## Completed — design (2026-07-09)

- TASK-025..035 (EPIC 04, full data model) → `data-model/01..11`
- TASK-049 (profile schema) → `data-model/02-users.md`
- TASK-121 (environment variables plan) → `ENVIRONMENT.md`
- Frontend integration contract v0.2 → `FRONTEND_INTEGRATION_PHASE.md`

## Completed — implementation, dev mode (2026-07-09)

| Tasks | Scope | Code |
|---|---|---|
| TASK-037..040 | Session strategy, RBAC enforcement, membership checks, denial behavior | `src/http.ts`, `src/rbac.ts`, `src/supabase.ts`, `src/errors.ts` |
| TASK-044..048 | Webhook signature validation, idempotency, subscription/entitlement sync, failed payment, cancellation | `src/stripe/*`, `src/routes/webhooks.ts`, `src/entitlements.ts` |
| TASK-042..043 | Checkout and customer portal flows | `src/routes/billing.ts` |
| TASK-050..053 | Profile self-edit fields, professor evolution (ranks/notes), append-only history, field-level permissions | `src/routes/profiles.ts`, `src/routes/students.ts` |
| TASK-058..061 | Attendance schema, QR generation, QR validation endpoint, occupancy | migrations 0007, `src/qr.ts`, `src/routes/sessions.ts`, `src/routes/checkin.ts` |
| TASK-065..068 | Feed posts, targeting visibility, pinning, reactions | migrations 0008, `src/routes/feed.ts` |
| TASK-073..077 | Academy, branch, room, modality, class schemas + onboarding | migrations 0002/0007, `src/routes/academies.ts` |
| TASK-081..084 | Invoices, immutable payments, delinquency rules, finance summary API | migrations 0006, `src/finance.ts`, `src/routes/finance.ts` |
| TASK-088..090 | Status history schema, student vs professor editable statuses | migrations 0004/0009, routes above |
| TASK-093..095 | Admin, professor, student dashboard APIs | `src/routes/dashboards.ts` |
| TASK-100..103 | Notification schema and feed/payment/attendance triggers | migrations 0008, `src/notify.ts` |
| TASK-106..109 | Audit schema; auth/billing/profile audit events | migrations 0009, `src/audit.ts` |
| TASK-116..118 | Backend, webhook, and auth/RBAC unit tests (23 passing) | `src/*.test.ts` |
| — | translation.ts contract (PO directive) | `shared/translation.ts` + tests |

Migrations 0001–0010 are written but **not applied** (no Supabase project exists).

## Partially blocked (need Marcia's EPIC 02 accounts)

- TASK-036 (Supabase Auth actually configured), TASK-041 (Stripe products actually created), TASK-122 (Supabase deployment prep) — code and design ready; provisioning needs the dev accounts and credentials (Hard Gate: only the PO handles credentials).
- End-to-end validation of every implemented flow (server runs, but no database/Stripe to run against).

## Remaining Claude tasks (not yet startable)

- TASK-117-style E2E webhook tests against Stripe test mode — after TASK-012.
- EPIC 18 deploy prep finalization — deploy itself is a Hard Gate.
