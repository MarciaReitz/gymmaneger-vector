# Integrations Design

Owner: Claude. **Design only.** Anything that touches Stripe, permissions, or access is a Hard Gate — no configuration or code until PO approval.

## Stripe (SaaS billing — academies pay the platform)

DRAFT product structure for TASK-041 — **awaiting PO approval, not yet a completed task**:

- One Stripe Product per SaaS tier (e.g. `Starter`, `Pro`) with monthly Prices in BRL.
- One Stripe Customer per academy (`academy_billing_customers.stripe_customer_id`).
- Checkout Session for purchase; Customer Portal for self-service management.
- Webhook events consumed (signature-validated, idempotent): `checkout.session.completed`, `customer.subscription.created|updated|deleted`, `invoice.paid`, `invoice.payment_failed`.
- Access entitlement changes only after webhook-confirmed events (source of truth rule).

## Student tuition (academies charge students)

MVP records tuition as first-class data (`plans`, `student_subscriptions`, `invoices`, `payments` — see data-model/07-billing.md). Whether student tuition is also collected through Stripe (Connect) is an open product decision for the PO — not assumed in MVP.

## Email

Transactional email provider (TASK-015, account owned by Marcia). Backend sends via provider API for: payment notifications, attendance confirmations. Design only.

## Monitoring

Error monitoring (TASK-018, account owned by Marcia) integrated at API layer. Design only.
