# Data Model: Family ToDo App

**Inputs**: Approved `spec.md` v9; approved constitution v1; submitted technical direction from Plan form `plan-technical-direction` (submission `05ff6061-5267-4bdd-8077-581dbf766f10`). The current `plan.md` v3 is a proposed draft ready for owner review; technical architecture details below are design proposals, not approved product decisions.

This is a proposed Supabase PostgreSQL model, not a migration. It has not been implemented or tested. Store identity in Supabase Auth; never store passwords or authentication secrets in application tables.

## Entities and relationships

```text
Auth User 1 ── 0..* Family Membership *..1 Family Space 1 ── 0..* Task
                                                       Task *── 1 Assignee Membership
```

- **Family Space** is the access boundary containing the one shared list for that space (FR-002).
- **Family Membership** connects an authenticated user to a family space and carries the user's role and displayed name. Only members of a space can access its shared list (FR-003, FR-012).
- **Task** belongs to one family space and exactly one responsible membership within that same space (FR-004).
- The initial admin must be established through the external owner/support process in FR-016; provisioning details are outside the in-app feature scope.
- Personal/member-specific lists are not modeled in v1 (FR-002).

## Proposed relational entities

### `family_spaces`

- `id uuid primary key`
- `created_at timestamptz not null` (implementation metadata; not a user-facing requirement)
- Optional `name text`: add only if the UX needs a family-space label; the feature does not require a naming workflow.

### `family_memberships`

- `id uuid primary key`
- `family_space_id uuid not null references family_spaces(id)`
- `user_id uuid not null references auth.users(id)`
- `display_name text not null` — needed to identify the assignee by name (FR-014). Define how this value is initially supplied and maintained during implementation; no profile-management workflow is specified.
- `role text not null check (role in ('admin', 'member'))`
- `created_at timestamptz not null` (implementation metadata; not a user-facing requirement)
- Unique (`family_space_id`, `user_id`) prevents duplicate membership.
- Unique (`family_space_id`, `id`) supports same-space task assignment integrity.

Membership and role mutations must only occur through authorized admin invite/add or external provisioning paths. A user must not be able to add themselves or elevate their own role. The invitation, acceptance, and account-identity association mechanics are not specified by the feature and remain open for implementation design.

### `tasks`

- `id uuid primary key`
- `family_space_id uuid not null references family_spaces(id)`
- `title text not null` — reject values that are empty or whitespace-only (FR-001, FR-010).
- `assignee_membership_id uuid not null`
- `priority text not null check (priority in ('High', 'Medium', 'Low'))` (FR-005, FR-011).
- `is_done boolean not null default false` (FR-008). Members may change completion state, including reopening a Done task, without changing other task fields.
- `created_at timestamptz not null`
- `updated_at timestamptz not null`
- Composite foreign key (`family_space_id`, `assignee_membership_id`) references `family_memberships` (`family_space_id`, `id`), preventing assignment outside the task's family space.
- Index by `family_space_id`; consider (`family_space_id`, `is_done`) if query plans warrant it. The default list query must include active and Done tasks (FR-015), not filter one state out.

This proposed shape does not prescribe a removal/soft-delete column. FR-007 requires admin removal from the shared list, while FR-009/SC-004 persistence applies to tasks not intentionally removed. Whether implementation physically deletes rows, marks them removed, or uses a separate retention mechanism is unresolved; do not infer recoverability, an audit-history requirement, or a retention period. Any selected mechanism must keep removed tasks out of the shared list and satisfy separately approved retention/security requirements. Avoid destructive cascade behavior until retention expectations are resolved.

## Invariants and access rules

1. A task in the shared list has one nonblank title, exactly one same-space assignee, one required priority, and a completion state (FR-001, FR-004–FR-005, FR-010–FR-011).
2. A user can read a family space, its memberships, and its shared tasks only if authenticated and a member of that space (FR-003, FR-012).
3. Admins can create, edit, and remove tasks; all members can change completion state (FR-013). Completion changes include setting Done and reopening to active, consistent with the specification's edge case.
4. A member's completion action may only change completion state and its update timestamp; it must not alter title, assignee, priority, or family-space ownership.
5. The default shared-list query includes Done alongside active tasks (FR-015).
6. The assignee's displayed name is the identifier; any per-person color is supplemental and never substitutes for priority (FR-014).
7. Persistence across leaving and revisiting applies to tasks that have not been removed under FR-007 (FR-009, SC-004).
8. An assignee must be a current membership in the same family space as the task; no cross-space assignment is permitted (FR-004).

## Supabase authorization design proposal

- Enable RLS on all application tables and default to deny when no policy grants access.
- Read policies must check `auth.uid()` against membership in the row's family space.
- Task insert, edit, and removal policies must require admin membership. Validate that a selected assignee belongs to the same space.
- Do not rely on row-level update policies alone to restrict changed columns. Implement completion via a narrowly scoped Postgres function/RPC or equivalent server-side operation that derives the caller from the authenticated session and updates only `is_done` and `updated_at` after membership checks.
- Keep service-role credentials server-only; never trust a client-supplied user ID for authorization.
- Do not permit arbitrary authenticated membership inserts. Choose a least-privilege invitation/provisioning mechanism consistent with the final flow, including the external initial-admin process.
- Add automated checks for cross-family isolation, admin/member action boundaries, same-space assignee integrity, non-removed task persistence, and removal from the shared list before release. This document is design guidance, not evidence that policies or tests exist or passed.

## Decisions deliberately left open

The feature does not specify whether users can belong to multiple family spaces; this relational model permits it without committing the UX to that behavior. Family-space naming, display-name maintenance, invitation acceptance and delivery, initial-admin provisioning, removal storage/cascade behavior, data retention, and account recovery require design decisions before production implementation. The email sign-in method is also unspecified. Do not treat this model's proposed Supabase/Auth and RLS shape as owner-approved implementation detail merely because the technical direction selected Supabase email/auth and database. Avoid destructive cascade behavior until retention expectations are defined.
