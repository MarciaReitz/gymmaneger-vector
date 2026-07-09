# Backend Decisions Log

Owner: Claude. Non-critical decisions recorded per Autonomous Build mode. Critical decisions (auth, billing, access, migrations, architecture conflicts) are Hard Gates and are **not** decided here.

| ID | Date | Decision | Rationale |
|---|---|---|---|
| D-001 | 2026-07-09 | Primary keys are `uuid` via `gen_random_uuid()` | Uniform IDs-only policy; safe for distributed generation |
| D-002 | 2026-07-09 | Multi-tenancy via `academy_id` column + RLS in a single shared schema | Simplest safe model for MVP scale; schema-per-tenant rejected as operational overhead |
| D-003 | 2026-07-09 | SaaS billing (academy → platform, Stripe) and tuition (student → academy, internal records) are separate domains | Prevents entitlement logic from mixing with academy revenue; Stripe webhook rule applies to SaaS domain |
| D-004 | 2026-07-09 | Current value + append-only history tables for all evolving fields (status, rank, profile fields) | Roadmap requires history without overwriting (TASK-052) |
| D-005 | 2026-07-09 | Backend task state tracked in `engine/STATE.md` | `roadmap/` is ChatGPT-owned; Claude must not edit it. ChatGPT should sync ROADMAP_MASTER from this ledger |
| D-006 | 2026-07-09 | Static catalogs (`roles`, `permissions`) carry a stable machine `key` for code use; all FKs still reference `id` | Code needs stable identifiers; IDs-only rule preserved for relationships |
| D-007 | 2026-07-09 | Enums as `text` + `check` constraint instead of Postgres enum types | Value lists evolve; avoids enum migration pain |
| D-008 | 2026-07-09 | No hard deletes anywhere; deactivation via `status` | Destructive operations are a Hard Gate; auditability |
| D-009 | 2026-07-09 | Design docs live in `engine/data-model/`; SQL migrations deferred to implementation phase | Migration apply is a Hard Gate; design must be approved first |
| D-010 | 2026-07-09 | Professor-managed evolution data kept out of `profiles` (separate `student_ranks`, `professor_notes`) | Keeps profile lightweight (TASK-049) and field-level permissions clean |
| D-011 | 2026-07-09 | Money as integer minor units + ISO 4217 currency | Standard practice; avoids float errors |
| D-012 | 2026-07-09 | Class/session tables designed inside TASK-032 scope, anticipating EPIC 10 FKs | Attendance FKs need stable targets; avoids rework |
