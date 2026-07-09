# Frontend Integration Phase

Owner: Claude (backend side of the contract). Consumer: Antigravity / Antenor (frontend). This document defines **what the backend exposes and guarantees** so the frontend can integrate without reading backend internals. Frontend implementation choices (components, state, styling) are entirely Antigravity's and are out of scope here.

Status: contract v0.2 — **implemented** in `gymmanager/backend/` (dev/test mode). Base URL in dev: `http://localhost:8787`. All endpoints below exist; auth is `Authorization: Bearer <supabase JWT>`.

**Mandatory translation layer:** every text, label, state, and display name in the UI resolves through `gymmanager/shared/translation.ts` (key→label catalogs; entity display names via the ID-based `DisplayNameResolver`). Never hardcode business names or status strings in components.

## 1. Integration model

The frontend talks to the backend through exactly two channels:

| Channel | Use | Rule |
|---|---|---|
| Supabase client (supabase-js) | Auth (sign-in/up/out, session refresh) and **read-only** queries on RLS-protected tables | RLS is the security boundary; the client never receives service-role keys |
| Backend API (Next.js route handlers, `/api/*`) | **All mutations** and any logic beyond a plain read (check-in, payments, posts, role grants) | Every handler re-validates session → membership → permission server-side |

Realtime (occupancy, feed updates) uses Supabase Realtime subscriptions on RLS-protected tables.

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

## 4. API surface (contract level)

Full request/response schemas will live in `engine/api/` as they are implemented; this table is the stable surface.

| Domain | Endpoints | Notes |
|---|---|---|
| Bootstrap | `GET /api/me` | session, memberships, roles, entitlements |
| Academy structure | `GET/POST/PATCH /api/academies/:id/branches`, `.../rooms`, `.../modalities`, `.../classes` | admin only; reads may also go via supabase-js |
| Profile | `GET /api/profiles/:id`, `PATCH /api/profiles/:id` (student fields), `GET /api/profiles/:id/history` | field-level permissions enforced server-side |
| Evolution | `POST /api/students/:id/ranks`, `POST /api/students/:id/notes` | professor permission `profile.evolution.edit` |
| Attendance | `GET /api/sessions?class_id=&from=&to=`, `POST /api/sessions/:id/attendance` (manual mark), `POST /api/checkin` `{ token }` (QR), `GET /api/sessions/:id/occupancy` | occupancy also via Realtime |
| Feed | `GET /api/feed?academy_id=` (targeting resolved server-side), `POST /api/posts`, `PATCH /api/posts/:id`, `POST /api/posts/:id/reactions` | |
| Finance (tuition) | `GET /api/finance/summary?academy_id=`, `GET /api/students/:id/invoices`, `POST /api/invoices/:id/payments` (manual record) | admin/support read; student sees own |
| SaaS billing | `POST /api/billing/checkout`, `POST /api/billing/portal`, `POST /api/webhooks/stripe` | webhook is backend-only; frontend just redirects to returned URLs |
| Dashboards | `GET /api/dashboards/admin`, `/professor`, `/student` (+ `academy_id`) | role-gated aggregates |
| Notifications | `GET /api/notifications`, `PATCH /api/notifications/:id/read` | |

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

- Breaking changes to any endpoint above require a version note in this file and coordination through Orquestra — never silent.
- The frontend must not depend on table names or columns beyond what RLS-protected reads expose for the read paths listed above.
- Mock data used by the frontend before backend readiness must match the shapes in this contract.

## 8. Open items

- ~~`app/` scaffold~~ Resolved (ASSUMPTIONS.md A-001/A-002): `gymmanager/frontend/` (Vite SPA, Antigravity), `gymmanager/backend/` (Hono service, Claude), `gymmanager/shared/` (contracts).
- Supabase/Stripe **dev** accounts (EPIC 02, Marcia) — required to run FI-1 end-to-end; only the PO handles credentials (Hard Gate).
