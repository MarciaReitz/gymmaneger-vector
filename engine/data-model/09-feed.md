# Feed (TASK-033)

Feed is official academy communication only (TASK-009/064) — no social-network behavior.

## post_categories

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK → academies | |
| name | text not null | display only |
| status | text not null | `active` / `archived` |
| created_at / updated_at | timestamptz | |

## posts

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK | |
| author_member_id | uuid FK → academy_members | resolves to user by ID |
| category_id | uuid FK → post_categories | |
| title | text not null | |
| body | text not null | |
| image_url | text | optional, single image (MVP media constraint) |
| pinned | boolean not null default false | TASK-067 |
| pinned_at | timestamptz | ordering among pinned |
| status | text not null | `draft` / `published` / `archived` |
| published_at | timestamptz | |
| created_at / updated_at | timestamptz | |

Indexes: `posts(academy_id, status, pinned desc, published_at desc)`.

## post_targets (TASK-066 visibility)

A post with no target rows is visible to the whole academy. With rows, visibility is the union of targets.

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK | |
| post_id | uuid FK → posts | |
| target_type | text not null | `branch` / `modality` / `class` / `role` |
| target_id | uuid not null | FK resolved per type at API layer; role targets use `roles.id` |
| created_at | timestamptz | |

Unique `(post_id, target_type, target_id)`.

## post_reactions (TASK-068)

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| academy_id | uuid FK | |
| post_id | uuid FK → posts | |
| member_id | uuid FK → academy_members | |
| reaction | text not null | `like` / `celebrate` / `support` (MVP set) |
| created_at | timestamptz | |

Unique `(post_id, member_id)` — one reaction per member per post (MVP).

## RLS

- Read: members matching the post's targeting. Create: members holding `feed.post.create` (admin, professor). Reactions: any active member on posts they can read.
