# Orquestra

This layer defines how the agents coordinate.

## Responsibilities

- define the agent roster
- define each agent's role and scope
- define ownership boundaries
- define hard gates and checkpoints
- prevent overlap between agents
- define shared contract files used between frontend and backend

## Files

- `AGENTS.md`
- `OWNERSHIP.md`
- `GATES.md`
- `CHECKPOINTS.md`
- `EXCEPTIONS.md`
- `CONFLICTS.md`
- `START.md`
- `ASSUMPTIONS.md`
- `DECISIONS.md`
- `translation.ts` rules for ID-based frontend i18n and lookup

## Notes

The frontend must use an ID-based translation layer. Visual text, labels, role names, and status strings should resolve through `translation.ts` or an equivalent contract, not hardcoded business names.
