# Audit and Event Logs (TASK-034)

Both tables are **immutable** (insert-only; no UPDATE/DELETE grants).

## audit_events

Every sensitive action is logged with actor and entity IDs.

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK, nullable | null for platform-level events |
| actor_user_id | uuid FK → user_accounts, nullable | null = system |
| action | text not null | machine key, e.g. `auth.login`, `role.granted`, `invoice.paid`, `profile.field.updated`, `subscription.status_changed` |
| entity_type | text not null | table/domain name |
| entity_id | uuid not null | |
| payload | jsonb | minimal diff/context; no secrets, no full PII dumps |
| created_at | timestamptz not null | |

Indexes: `audit_events(academy_id, created_at desc)`, `audit_events(entity_type, entity_id)`, `audit_events(actor_user_id)`.

Audited domains (minimum): auth events (TASK-107), role/permission grants, billing and subscription changes (TASK-108), profile edits (TASK-109), attendance corrections, admin grants/revocations.

## webhook_events

Inbound provider events; the idempotency ledger for Stripe.

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| provider | text not null | `stripe` |
| provider_event_id | text not null | unique with provider |
| event_type | text not null | |
| payload | jsonb not null | raw event |
| status | text not null | `received` / `processed` / `failed` / `skipped` |
| processed_at | timestamptz | |
| error_detail | text | |
| created_at | timestamptz | |

Unique `(provider, provider_event_id)` — duplicate deliveries are no-ops.

## RLS

- `audit_events`: academy `admin` reads own academy's events; no client writes (service role only).
- `webhook_events`: service role only.
