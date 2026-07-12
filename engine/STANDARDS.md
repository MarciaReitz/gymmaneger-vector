# Backend Implementation Standards

Owner: Claude.

## Naming

- Tables: `snake_case`, plural (`students`, `attendance_records`).
- Columns: `snake_case`. Foreign keys: `<entity>_id` (`academy_id`, `student_id`).
- Enums: implemented as `text` columns constrained by `check` against a documented value list (see DECISIONS.md D-007).

## Mandatory columns

Every table has:

- `id uuid primary key default gen_random_uuid()`
- `created_at timestamptz not null default now()`
- `updated_at timestamptz not null default now()` (maintained by trigger; omitted on immutable tables)

Every tenant-scoped table additionally has `academy_id uuid not null references academies(id)`.

## Data rules

- **IDs only.** No business logic may match on names, emails, or any mutable attribute. Relationships and lookups use IDs.
- **No hard deletes.** Rows are deactivated via `status` columns. Destructive operations are a Hard Gate.
- **Immutable tables** (`payments`, `audit_events`, `profile_field_history`, `professor_notes`, `webhook_events`): insert-only. No `UPDATE`/`DELETE` grants; enforced by privileges and RLS.
- **History over overwrite.** Fields with evolution semantics (belt, weight, status) keep the current value in the parent table and every change in a history table with author and timestamp.
- **Money** is stored as integer minor units (`amount_cents bigint`) plus `currency char(3)` (ISO 4217). Never floats.
- **Timestamps** are always `timestamptz` (UTC).

## API standards

- All handlers validate: authenticated session → academy membership by ID → role permission → then execute.
- Webhook handlers are idempotent and keyed by provider event ID.
- Errors return stable machine-readable codes; no internal details leak to clients.
