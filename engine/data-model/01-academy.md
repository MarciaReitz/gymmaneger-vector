# Academy Structure (TASK-025)

## academies

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| display_name | text not null | Display only, never a key |
| legal_name | text | |
| status | text not null | `active` / `suspended` / `archived` |
| created_at / updated_at | timestamptz | |

## branches

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK → academies | tenant key |
| name | text not null | display only |
| address_line1 / address_line2 / city / state / postal_code / country | text | |
| status | text not null | `active` / `inactive` |
| created_at / updated_at | timestamptz | |

## rooms

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK → academies | denormalized tenant key for RLS |
| branch_id | uuid FK → branches | |
| name | text not null | display only |
| capacity | integer not null check (capacity > 0) | drives occupancy (EPIC 08) |
| status | text not null | `active` / `inactive` |
| created_at / updated_at | timestamptz | |

## modalities (extensible catalog, supports TASK-076)

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK → academies | per-academy catalog |
| name | text not null | display only (e.g. shown as "Jiu-Jitsu") |
| status | text not null | `active` / `inactive` |
| created_at / updated_at | timestamptz | |

## Indexes

- `branches(academy_id)`, `rooms(academy_id)`, `rooms(branch_id)`, `modalities(academy_id)`

## RLS

- Read: members of the academy. Write: `admin` role of the academy.
