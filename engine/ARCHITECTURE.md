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

## Multi-tenancy

- Tenant unit: **academy**. Every tenant-scoped table carries `academy_id uuid not null` referencing `academies.id`.
- Isolation is enforced by Postgres Row Level Security on every tenant table. Policies validate membership through `academy_members` by ID.
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

Implementation of this flow (EPIC 05, EPIC 06) is behind Hard Gates (auth, billing) and requires PO approval before any code or Stripe configuration.

## Environments

- `local` — developer machine, Supabase local or dev project
- `staging` — pre-production, separate Supabase project and Stripe test mode
- `production` — live, Stripe live mode

Migration apply, deploy, and production data mutation are Hard Gates in all environments above local.
