# Environment Variables Plan (TASK-121)

Owner: Claude. This is the plan only — no secrets are stored here or anywhere in the repositories. Canonical contract fixed by PO directive (Sprint 2); working reference: `gymmanager/backend/.env.example` + `SETUP.md`.

## Backend (server-only)

| Variable | Purpose |
|---|---|
| `SUPABASE_URL` | Supabase project URL |
| `SUPABASE_ANON_KEY` | Validates user JWTs (identity only — never used for data) |
| `SUPABASE_SERVICE_ROLE` | Privileged persistence access (repository layer only) |
| `DATABASE_URL` | Direct Postgres connection (Prisma tooling / migrations) |
| `STRIPE_SECRET_KEY` | Stripe API (server) |
| `STRIPE_WEBHOOK_SECRET` | Webhook signature validation |
| `APP_URL` | Frontend origin (CORS) |
| `PORT` | HTTP port (default 8787) |
| `DEV_ENTITLEMENT_BYPASS` | Dev-only; never set in production |

`SUPABASE_SERVICE_ROLE_KEY` is accepted as a legacy alias for `SUPABASE_SERVICE_ROLE`.

## Frontend (public, Vite)

With API First the frontend needs only Supabase Auth (identity) and the API base URL — no data keys:

| Variable | Purpose |
|---|---|
| `VITE_SUPABASE_URL` | Supabase Auth |
| `VITE_SUPABASE_ANON_KEY` | Supabase Auth (public anon key) |
| `VITE_API_URL` | Backend base URL (`/api/v1`) |

## Rules

- Separate values per environment (local / staging / production); staging uses Stripe test mode.
- Secrets live only in the hosts' secret stores, never in git; only the PO handles credentials (Hard Gate).
- Rotation of any production secret is a deploy-adjacent operation → Hard Gate.
- Email provider and monitoring variables will be added when those integrations enter scope (TASK-015/018).
