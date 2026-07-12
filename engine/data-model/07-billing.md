# Billing (TASK-031)

**Design only.** Billing/payment implementation and any Stripe configuration are Hard Gates.

Two separated domains (DECISIONS.md D-003):

- **SaaS billing** — academies pay the platform (Stripe, webhook = source of truth).
- **Tuition** — academies charge students (first-class data in MVP; Stripe Connect is an open PO decision).

## SaaS billing

### academy_billing_customers

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK → academies, unique | |
| stripe_customer_id | text not null unique | external ID reference |
| created_at / updated_at | timestamptz | |

### academy_subscriptions

Written **only** by the webhook handler.

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK → academies | |
| stripe_subscription_id | text not null unique | |
| stripe_price_id | text not null | maps to SaaS tier |
| status | text not null | mirror of Stripe status: `trialing` / `active` / `past_due` / `canceled` / `unpaid` |
| current_period_end | timestamptz | |
| canceled_at | timestamptz | |
| created_at / updated_at | timestamptz | |

Entitlement rule: academy has paid access iff latest subscription status ∈ (`trialing`, `active`). Failed payment → `past_due` → grace behavior defined by PO at EPIC 06 (gated).

## Tuition

### plans (per academy)

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK → academies | |
| name | text not null | display only |
| amount_cents | bigint not null check (> 0) | |
| currency | char(3) not null | ISO 4217, default `BRL` |
| interval | text not null | `monthly` (MVP) |
| status | text not null | `active` / `archived` |
| created_at / updated_at | timestamptz | |

### student_subscriptions

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK | |
| student_id | uuid FK → students | |
| plan_id | uuid FK → plans | |
| status | text not null | `active` / `paused` / `canceled` |
| started_at | timestamptz not null | |
| ended_at | timestamptz | |
| created_at / updated_at | timestamptz | |

Unique: one `active` subscription per student (partial index).

### invoices

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK | |
| student_id | uuid FK → students | |
| subscription_id | uuid FK → student_subscriptions | |
| amount_cents | bigint not null | snapshot at issue time |
| currency | char(3) not null | |
| due_date | date not null | |
| status | text not null | `open` / `paid` / `overdue` / `void` |
| created_at / updated_at | timestamptz | |

Indexes: `invoices(academy_id, status, due_date)`, `invoices(student_id)`.

### payments (immutable)

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK | |
| invoice_id | uuid FK → invoices | |
| amount_cents | bigint not null check (> 0) | |
| currency | char(3) not null | |
| method | text not null | `cash` / `pix` / `card` / `transfer` / `other` |
| recorded_by_user_id | uuid FK → user_accounts | |
| paid_at | timestamptz not null | |
| created_at | timestamptz | insert-only; corrections via reversal rows, never edits |

## Delinquency (TASK-083 input)

`overdue` = invoice past `due_date` and not `paid`/`void`. Student status transitions driven by the status engine (EPIC 12) with reason codes.

## RLS

- Students read their own invoices/payments; admin (and `support` read-only) manage academy finance; SaaS tables readable by academy `owner`/`admin`, writable only by the webhook service role.
