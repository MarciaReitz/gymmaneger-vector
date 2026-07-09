# gymmaneger-vector

This repository is the Vector operating base for the martial arts academy application.

## Purpose

It defines how the agents coordinate, where each agent works, and how the project is organized into three layers:

- **Core / Orquestra**: agent coordination, roles, boundaries, checkpoints, and gates.
- **Engine**: technical architecture, implementation standards, data rules, security, and integrations.
- **Roadmap**: phases, priorities, tasks, and project state.

## Agent model

- **ChatGPT**: orchestrator
- **Claude**: backend engineer
- **Antigravity / Antenor**: frontend engineer

## Operating rules

- All domain logic must be based on IDs, never on names.
- Permissions, access control, and billing changes are hard gates.
- Stripe webhooks are the source of truth for paid access.
- Each zone has a single owner.
- Agents must not cross ownership boundaries without explicit approval.

## Folder map

- `orquestra/` — coordination rules and agent ownership
- `engine/` — technical architecture and implementation standards
- `roadmap/` — delivery state, phases, and next actions

## Entry point for agents

All agents should start by reading:

- `orquestra/AGENTS.md`
- `orquestra/OWNERSHIP.md`
- `orquestra/GATES.md`
- `engine/ARCHITECTURE.md`
- `roadmap/MASTER_STATE.md`
- `roadmap/NEXT_ACTIONS.md`
