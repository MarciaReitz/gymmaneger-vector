# Backend Technical Architecture

Owner: Claude. Scope: backend technical architecture only. Product-level system architecture (TASK-019, owner: ChatGPT) remains pending and takes precedence if it conflicts with this document.

## Stack

| Layer | Technology | Rationale |
|---|---|---|
| Database | Supabase Postgres | Managed Postgres with RLS, matches TASK-011 |
| Auth | Supabase Auth | JWT + refresh sessions, OAuth-ready |
| API | Standalone TypeScript service (Hono on Node) | Frontend is a Vite SPA (`gymmanager/frontend/`), so the API is a separate service (`gymmanager/backend/`); D-015 supersedes the earlier Next.js plan |
| Billing | Stripe | Webhook is the source of truth for paid access |
| Storage | Supabase Storage | Profile photo only (MVP constraint) |

## API First (PO directive, 2026-07-09)

**All frontend↔data communication goes exclusively through the Backend API.** Supabase is a persistence layer; the frontend never accesses the database directly (sole exception: Supabase Auth as identity provider). The backend is layered:

| Layer | Location | Responsibility |
|---|---|---|
| API Gateway | `backend/src/gateway/` + `app.ts` | HTTP entry point under `/api/v1`; auth middleware, zod validation, CORS, logging, stable error envelope |
| Services | `backend/src/services/` | Business logic; membership + permission checks; audit/notification side effects |
| Repositories | `backend/src/repositories/` | The ONLY layer that talks to Supabase |

Import rules: gateway → services → repositories, never skipping or reversing. Contracts are versioned (`shared/contracts/v1.ts`, OpenAPI at `backend/openapi/openapi.v1.yaml`, Swagger UI at `/api/v1/docs`); breaking changes require `/api/v2`. Frontend handoff: `gymmanager/HANDOFF.md`.

## Multi-tenancy

- Tenant unit: **academy**. Every tenant-scoped table carries `academy_id uuid not null` referencing `academies.id`.
- Isolation is enforced server-side by the service layer (membership by ID on every request). Postgres RLS remains enabled on every tenant table as **defense in depth** — with API First the anon key is never used for data, so RLS is the safety net, not the primary boundary.
- Single database, shared schema. Schema-per-tenant was rejected (see DECISIONS.md D-002).

## Identity and keys

- All primary keys are `uuid` (generated with `gen_random_uuid()`).
- All relationships use IDs. Names are display data only and are never used as relational or business keys.
- Stable machine keys (e.g. `roles.key`, `permissions.key`) exist only as code-facing identifiers for static catalogs; all foreign keys still reference `id`.

## Access entitlement flow (design)

1. Academy subscribes to a SaaS plan via Stripe Checkout.
2. Stripe sends webhook events to a validated endpoint (signature checked before processing).
3. The webhook handler records the event in `webhook_events` (idempotent by provider event ID) and updates `academy_subscriptions`.
4. Entitlements are derived exclusively from webhook-confirmed subscription state. No client- or UI-initiated write can grant paid access.

Implemented in dev/test mode (services/webhook.ts, services/entitlement.ts) per GATES.md Build Mode. Live Stripe configuration and production billing remain Hard Gates.

## Environments

- `local` — developer machine, Supabase local or dev project
- `staging` — pre-production, separate Supabase project and Stripe test mode
- `production` — live, Stripe live mode

Migration apply, deploy, and production data mutation are Hard Gates in all environments above local.
