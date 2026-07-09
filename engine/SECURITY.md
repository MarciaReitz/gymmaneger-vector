# Security Design

Owner: Claude. **This document is design only.** Any implementation or change of auth, access control, or permissions is a Hard Gate and requires PO approval first.

## Roles (from approved user types, TASK-008)

| Role key | Scope | Summary |
|---|---|---|
| `admin` | academy | Full management of one academy |
| `professor` | academy | Classes, attendance, student evolution fields |
| `student` | academy | Own profile, own billing, check-in, feed read |
| `support` | academy | Read/limited assist access for academy staff |

Roles are assigned per academy through `user_roles` (`user_id` + `academy_id` + `role_id`). A user may hold different roles in different academies.

## Enforcement layers

1. **Supabase Auth** — identity, JWT, refresh sessions.
2. **RLS** — every tenant table checks academy membership by ID; role-sensitive tables additionally check role.
3. **API layer** — permission checks server-side before any mutation (client is never trusted).

## Session strategy (design)

- Short-lived JWT access tokens; refresh tokens rotated on use.
- Session revocation on role change or membership end.

## Access denial behavior (design, TASK-040)

- Unauthenticated → redirect to login.
- Authenticated without membership in the target academy → blocked with a clean error; no data leakage about the academy's existence.
- Authenticated member without permission → 403 with stable error code.

## Paid access

Paid access is derived only from webhook-confirmed Stripe state (see ARCHITECTURE.md). UI state never grants access.
