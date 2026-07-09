# Backend Assumptions

Owner: Claude. Working assumptions made during Autonomous Build; each one is reversible. The PO can veto any of them — flag it and the code/docs will be adjusted.

| ID | Date | Assumption | Basis |
|---|---|---|---|
| A-001 | 2026-07-09 | `gymmanager` is the application repo: `frontend/` (Antigravity), `backend/` (Claude), `shared/` (contracts, named owner per file) | Frontend scaffold appeared in `gymmanager/frontend/` |
| A-002 | 2026-07-09 | Backend is a standalone Hono/Node service (port 8787); Vite SPA (port 5173) consumes it with bearer JWT | Frontend is Vite, not Next.js — supersedes the Next.js route-handler plan |
| A-003 | 2026-07-09 | `shared/translation.ts` is the canonical translation contract; owner: Claude; Antigravity consumes it | PO directive + orquestra README notes |
| A-004 | 2026-07-09 | pt-BR is the only MVP locale | Product language; contract supports adding locales later |
| A-005 | 2026-07-09 | `DEV_ENTITLEMENT_BYPASS=true` treats academies without a Stripe subscription as ACTIVE in dev only | Stripe account (TASK-012, Marcia) does not exist yet; production keeps webhook-only rule |
| A-006 | 2026-07-09 | Feed `branch` targeting behaves as academy-wide in MVP | No branch-membership model exists yet; revisit when it does |
| A-007 | 2026-07-09 | Invoices are issued via API/manually in MVP; overdue status computed on read and re-derived on payment | No job scheduler in MVP scope |
| A-008 | 2026-07-09 | Migration apply in the DEV Supabase project is authorized (non-destructive, CREATE-only); staging/production apply remains a Hard Gate | GATES.md Build Mode |
