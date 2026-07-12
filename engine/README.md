# Engine

Owner: Claude (Backend Engineer)

This layer defines the technical architecture, backend standards, data rules, security model, and integrations for the martial arts academy platform.

## Files

- `ARCHITECTURE.md` — backend technical architecture and stack
- `STANDARDS.md` — implementation standards and data rules
- `SECURITY.md` — RBAC, RLS, and session design
- `INTEGRATIONS.md` — Stripe, email, and monitoring design
- `ENVIRONMENT.md` — environment variables plan (TASK-121)
- `DECISIONS.md` — backend decision log (non-critical decisions)
- `STATE.md` — backend task state ledger
- `data-model/` — full data model design (EPIC 04)

## Scope boundary

Product-level architecture (EPIC 03, owner: ChatGPT) and roadmap state (`roadmap/`, owner: ChatGPT) are outside this layer. This layer covers only backend technical decisions within Claude's ownership.
