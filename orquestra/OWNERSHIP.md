# Ownership

## Purpose

Defines which agent may edit which area.

## Ownership map

| Area | Owner | Notes |
|---|---|---|
| `orquestra/` | ChatGPT | Coordination, rules, gates, agent registry. |
| `engine/` | Claude | Technical architecture, backend standards, integrations. |
| `roadmap/` | ChatGPT | State, phases, priorities, next actions. |
| `app/` | Claude + Antigravity/Antenor | Claude owns backend zones; Antigravity/Antenor owns frontend zones. |

## Rules

- One owner per area.
- Forbidden areas must never be edited directly.
- Shared files require a named owner.
- If ownership is unclear, stop and escalate.
