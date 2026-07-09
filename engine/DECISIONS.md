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
| D-013 | 2026-07-09 | Backend↔frontend contract lives in `engine/FRONTEND_INTEGRATION_PHASE.md`; ~~two channels~~ **channel model superseded by D-019 (API First — single API channel)** | Contract location still valid; canonical onboarding now `gymmanager/HANDOFF.md` |
| D-014 | 2026-07-09 | Stable machine error codes in a fixed envelope for all API errors | Frontend can branch on codes; no internal detail leakage |
| D-015 | 2026-07-09 | Backend implemented as standalone Hono/Node service in `gymmanager/backend/` | Frontend turned out to be a Vite SPA, not Next.js; supersedes the route-handler plan in ARCHITECTURE.md v1 |
| D-016 | 2026-07-09 | Translation contract implemented at `gymmanager/shared/translation.ts` (key→label catalogs + ID-based DisplayNameResolver) | PO directive: mandatory resolution layer for all UI text; IDs-only rule extended to display names |
| D-017 | 2026-07-09 | QR tokens: 10-min TTL, only SHA-256 hash persisted, backend-only validation | Prevents token leakage via database reads; frontend never sees stored tokens |
| D-018 | 2026-07-09 | ~~RLS grants frontend read-only access; mutations via API~~ **Superseded by D-019 (API First)** | Kept for history |
| D-019 | 2026-07-09 | API First implemented per PO directive: backend layered as Gateway (`gateway/`) → Service (`services/`) → Repository (`repositories/`); repositories are the only Supabase callers; frontend has no data access of any kind | Enforces the "frontend never touches the database" rule structurally; services testable with fake repos |
| D-020 | 2026-07-09 | RLS demoted from primary boundary to defense-in-depth; authorization is server-side in the service layer (membership + permission by ID per request) | With API First the anon key never reads data; policies stay enabled as a safety net |
| D-021 | 2026-07-09 | Contracts versioned by URL prefix: `/api/v1` + `shared/contracts/v1.ts` + `openapi.v1.yaml`; additive = same version, breaking = new version, log in contracts CHANGELOG | Predictable evolution for the frontend; no silent breaking changes |
| D-022 | 2026-07-09 | OpenAPI spec hand-maintained in YAML and served with Swagger UI at `/api/v1/docs`; zod schemas in the gateway remain the runtime validators | Lightweight now; can migrate to zod-openapi generation later without contract change |
| D-023 | 2026-07-09 | Realtime features (occupancy) use API polling in MVP; no direct DB subscriptions from the client | Direct Supabase Realtime would violate API First; push channel can be added behind the API later |
