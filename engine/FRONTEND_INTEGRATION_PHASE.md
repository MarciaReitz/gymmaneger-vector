# Frontend Integration Phase

Owner: Claude (backend side of the contract). Consumer: Antigravity / Antenor (frontend). This document defines **what the backend exposes and guarantees** so the frontend can integrate without reading backend internals. Frontend implementation choices (components, state, styling) are entirely Antigravity's and are out of scope here.

Status: contract **v1 (API First)** — implemented in `gymmanager/backend/` (dev/test mode). Base URL in dev: `http://localhost:8787/api/v1`. Swagger UI: `/api/v1/docs`. Canonical onboarding doc: **`gymmanager/HANDOFF.md`**; typed contracts: `gymmanager/shared/contracts/v1.ts`.

**Mandatory translation layer:** every text, label, state, and display name in the UI resolves through `gymmanager/shared/translation.ts` (key→label catalogs; entity display names via the ID-based `DisplayNameResolver`). Never hardcode business names or status strings in components.

## 1. Integration model (API First — PO directive 2026-07-09)

**The frontend never accesses the database.** Supabase is a persistence layer owned by the backend; there is exactly one data channel:

| Channel | Use | Rule |
|---|---|---|
| Supabase Auth (supabase-js) | Sign-in/up/out and session refresh ONLY — identity provider, not data | Public anon key is used for Auth exclusively; data queries via supabase-js are forbidden |
| Backend API (`/api/v1/*`) | **Everything else** — every read and every mutation | Gateway → Service → Repository; every request re-validates session → membership → permission server-side |

Realtime niceties (live occupancy) are handled by polling `GET /sessions/:id/occupancy` in MVP; a push channel, if ever added, will be exposed by the backend — never by direct DB subscriptions.

## 2. Identity and session contract

- Session comes from Supabase Auth (JWT + rotating refresh). The frontend never stores credentials or builds its own session logic.
- After login, the frontend bootstraps with `GET /api/me`, which returns:
  - `user` (id, profile summary)
  - `memberships[]` — one entry per academy: `{ academy_id, member_id, status, roles: [role keys], student_id? , professor_id? }`
  - `entitlement` per academy: `{ status: ACTIVE | TRIALING | PAST_DUE | LOCKED }`
- **IDs only**: every route and payload references entities by uuid. Names are display data.
- The active academy context is always an explicit `academy_id` — sent by the frontend on every API call and query.

## 3. Entitlement states the frontend must render

Derived exclusively from webhook-confirmed Stripe state (never from client-side calculation):

| State | Frontend behavior |
|---|---|
| `ACTIVE` / `TRIALING` | Full app |
| `PAST_DUE` | Full app + persistent billing warning banner |
| `LOCKED` | Billing/lock screen only; navigation blocked |

## 4. API surface (contract v1)

The authoritative, always-current surface is the OpenAPI spec (`backend/openapi/openapi.v1.yaml`, rendered at `/api/v1/docs`) plus the typed contracts (`shared/contracts/v1.ts`). Summary (all paths relative to `/api/v1`):

| Domain | Endpoints | Notes |
|---|---|---|
| Bootstrap | `GET /me` | session, memberships, roles, permissions, entitlements |
| Academy structure | `POST /academies`, `POST /academies/{branches,rooms,modalities,classes}` | admin only |
| Profile | `GET/PATCH /profiles/:id`, `GET /profiles/:id/history` | field-level permissions enforced server-side |
| Evolution | `POST /students/:id/ranks`, `POST /students/:id/notes` | professor permission `profile.evolution.edit` |
| Attendance | `GET/POST /sessions`, `POST /sessions/:id/qr`, `POST /sessions/:id/attendance`, `GET /sessions/:id/occupancy`, `POST /checkin` | occupancy by polling in MVP |
| Feed | `GET /feed?academy_id=` (targeting resolved server-side), `POST /feed/posts`, `PATCH /feed/posts/:id/pin`, `POST /feed/posts/:id/reactions` | |
| Finance (tuition) | `GET /finance/summary?academy_id=`, `POST /finance/plans`, `POST /finance/invoices`, `POST /finance/invoices/:id/payments`, `GET /students/:id/invoices` | admin/support read; student sees own |
| SaaS billing | `POST /billing/checkout`, `POST /billing/portal`, `POST /webhooks/stripe` | webhook is backend-only; frontend just redirects to returned URLs |
| Dashboards | `GET /dashboards/{admin,professor,student}?academy_id=` | role-gated aggregates |
| Notifications | `GET /notifications`, `PATCH /notifications/:id/read` | |

Contract versioning: additive changes stay in v1; breaking changes ship as `/api/v2` + `contracts/v2.ts` with both versions running during migration (`shared/contracts/CHANGELOG.md`).

## 5. Error contract

All errors: HTTP status + `{ "error": { "code": "<STABLE_CODE>", "message": "<safe text>" } }`.

Stable codes the frontend must handle: `UNAUTHENTICATED` (401), `FORBIDDEN` (403), `NOT_FOUND` (404), `VALIDATION_FAILED` (422), `CONFLICT` (409), `SUBSCRIPTION_INACTIVE` (403 + lock screen), `QR_INVALID` / `QR_EXPIRED` (400), `RATE_LIMITED` (429). No internal details ever leak in `message`.

## 6. Integration sequence (proposed phases)

| Phase | Scope | Backend prerequisite |
|---|---|---|
| FI-1 | Auth + app shell + `GET /api/me` + role-based navigation | EPIC 05 (dev mode) |
| FI-2 | Profile views (summary, edit, history) — TASK-054..056 | EPIC 07 backend |
| FI-3 | Finance dashboard + student billing views — TASK-085..086 | EPIC 11 backend |
| FI-4 | Attendance list + QR check-in — TASK-062..063 | EPIC 08 backend |
| FI-5 | Feed (list, composer, detail) — TASK-069..071 | EPIC 09 backend |
| FI-6 | Dashboards + notifications — TASK-096..098, 104 | EPICs 13–14 backend |

Sequencing of frontend tasks is orchestration (ChatGPT); this table only states backend readiness order.

## 7. Contract stability rules

- Breaking changes to any endpoint above require a new API version (`/api/v2`) and coordination through Orquestra — never silent.
- The frontend must not depend on database table names or columns at all — only on the shapes in `shared/contracts/v1.ts`.
- Mock data used by the frontend before backend readiness must match the shapes in `shared/contracts/v1.ts`.

## 8. Open items

- ~~`app/` scaffold~~ Resolved (ASSUMPTIONS.md A-001/A-002): `gymmanager/frontend/` (Vite SPA, Antigravity), `gymmanager/backend/` (Hono service, Claude), `gymmanager/shared/` (contracts).
- Supabase/Stripe **dev** accounts (EPIC 02, Marcia) — required to run FI-1 end-to-end; only the PO handles credentials (Hard Gate).
