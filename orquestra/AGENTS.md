# Agents

## Purpose

Registry of the project's AI agents.

## Agents

| Agent | Role | Scope |
|---|---|---|
| ChatGPT | Orchestrator | Coordinates work, updates state, resolves sequencing, enforces gates. |
| Claude | Backend Engineer | API, database, auth, RBAC, payments, webhooks, business logic. |
| Antigravity / Antenor | Frontend Engineer | UI, components, flows, client state, presentation layer. |

## Rules

- Each agent has one role and one scope.
- No two agents own the same area.
- Scope overlaps must be resolved through Orquestra, not by direct edits.
