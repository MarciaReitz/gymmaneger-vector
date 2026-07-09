# Gates

## Hard gates

The agent must stop and wait for human approval for:

- merge to protected branches
- deploy
- migration apply
- production data mutation
- auth or access-control changes
- billing or payment changes
- destructive operations
- ownership conflicts
- architecture decisions
- uncertainty

## Not a stop condition

- draft PR
- report
- completed task or sprint
- natural pause

## Rule

If the change affects Stripe, permissions, or access, it is a hard gate.
