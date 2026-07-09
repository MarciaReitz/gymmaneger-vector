# Environment Variables Plan (TASK-121)

Owner: Claude. This is the plan only — no secrets are stored here or anywhere in the repositories.

## Secrets (server-only, never exposed to the client)

| Variable | Purpose |
|---|---|
| `SUPABASE_SERVICE_ROLE_KEY` | Server-side privileged database access |
| `SUPABASE_JWT_SECRET` | JWT verification |
| `STRIPE_SECRET_KEY` | Stripe API (server) |
| `STRIPE_WEBHOOK_SECRET` | Webhook signature validation |
| `EMAIL_PROVIDER_API_KEY` | Transactional email |
| `MONITORING_DSN` | Error monitoring |

## Public (safe for client bundle)

| Variable | Purpose |
|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Public anon key (RLS-protected) |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | Stripe.js |
| `NEXT_PUBLIC_APP_URL` | Canonical app URL |

## Rules

- Separate values per environment (local / staging / production); staging uses Stripe test mode.
- Secrets live only in Vercel/Supabase environment configuration, never in git.
- Rotation of any production secret is a deploy-adjacent operation → Hard Gate.
