# Gates

## Purpose

Defines the moments an agent must stop and wait for human approval.

## Build mode

During the bootstrap phase, agents may build autonomously inside their ownership areas without waiting for missing documentation, as long as the work does not cross a hard gate.

Missing files owned by the same agent are not blockers. The agent should create them.

Auth, RBAC, Stripe, webhooks, migrations, and backend APIs may be implemented in development or test mode without stopping, as long as no production change is involved.

## Hard gates

An agent must stop for approval when the work involves:

- merge to a protected branch
- deploy to production or staging with release risk
- migration apply with destructive or irreversible impact
- production data mutation
- production auth or access-control changes affecting real users
- production billing or payment changes
- destructive operations such as delete, drop, overwrite, or force-push
- ownership conflicts
- explicit architecture decisions that affect another agent's area
- uncertainty after reasonable investigation
- no eligible work left in the agent's ownership area
- continuous-work limit reached

## Not a stop condition

These are checkpoints, not stop conditions:

- opening a draft PR
- writing or updating documentation
- creating missing files in the agent's own ownership area
- finishing a sprint, task, or epic
- reaching a natural pause
- implementing auth, RBAC, Stripe, webhooks, or backend APIs in test/dev mode

## Rule

If the change affects production Stripe, production permissions, production authentication, access control, or production data, it is a hard gate.
