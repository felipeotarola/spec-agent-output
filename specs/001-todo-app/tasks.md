# Tasks: Family ToDo App

**Input**: Approved `specs/001-todo-app/spec.md` v9, `plan.md` v3, `data-model.md` v3, and approved constitution v1.
**Status**: Draft task plan. Every implementation task is unchecked; no coding or test execution is claimed.
**Source layout**: Paths below follow the proposed layout in `plan.md` § Project Structure. The repository is not selected or inspected in this workspace; reconcile paths and conventions with the actual repository before implementation.
**Technical choices**: Follow the owner-submitted direction (Next.js, React, TypeScript, Supabase database and email-based auth). Select exact package versions and test runner during setup. Do not infer a password/OTP/magic-link UX, invitation-acceptance flow, external initial-admin provisioning workflow, or removed-task retention policy; see deferred implementation Questions.

## Format

`[ID] [P?] [Story] Description`

- `[P]` means independent work with no overlapping dependency.
- Story labels map implementation work to the approved spec's user stories. Foundational tasks are shared prerequisites.
- All coding and test tasks remain unchecked until a coding agent performs them and reports evidence.

## Phase 1: Setup

**Purpose**: Confirm the implementation workspace and establish the selected stack.

- [ ] T001 Inspect the selected repository and reconcile its conventions with proposed source paths in `plan.md` § Project Structure; record any required path changes before implementation.
- [ ] T002 Initialize or align the TypeScript/Next.js application and React dependencies in `package.json` and the repository's selected configuration files; pin versions compatible with the repository and current supported stack.
- [ ] T003 [P] Configure linting, formatting, TypeScript checks, and the selected test runner in repository configuration files; document the chosen commands in `package.json`.
- [ ] T004 [P] Create the planned application, Supabase migration, database-test, and application-test directories if they do not already exist: `src/app/`, `src/components/`, `src/features/family/`, `src/features/tasks/`, `src/lib/supabase/`, `src/lib/validation/`, `src/types/`, `supabase/migrations/`, `supabase/tests/`, and `tests/{unit,components,integration}/`.
- [ ] T005 Add environment-variable names and safe local setup instructions to `.env.example` and `README.md`; do not place credentials in committed files or expose service-role credentials to browser code.

## Phase 2: Foundation

**Purpose**: Establish persistence, identity, and family-space authorization needed before user-story work.

- [ ] T006 Create the initial Supabase migration for `family_spaces`, `family_memberships`, and `tasks` in `supabase/migrations/`; include keys, same-space assignee integrity, role and priority constraints, nonblank-title enforcement, completion state, and timestamps consistent with `data-model.md`.
- [ ] T007 [P] Create server-side and browser Supabase client/session helpers in `src/lib/supabase/`; ensure privileged credentials remain server-only.
- [ ] T008 Enable RLS and add least-privilege family-membership-scoped read policies for application tables in `supabase/migrations/`; deny access to nonmembers and prevent self-added membership or self-elevation.
- [ ] T009 Implement and test the narrowly scoped authenticated completion operation in `supabase/migrations/` and `src/features/tasks/`; derive identity from the authenticated session and allow changing completion state without changing title, assignee, priority, or family-space ownership.
- [ ] T010 Add automated database authorization tests in `supabase/tests/` for cross-family isolation, member/admin boundaries, same-space assignee integrity, and completion-only updates.
- [ ] T011 Add shared task-input validation in `src/lib/validation/` for nonblank title, exactly one valid same-space assignee, and required High/Medium/Low priority; add unit tests in `tests/unit/`.
- [ ] T012 Establish safe authenticated route/layout boundaries and access-denied/error handling in `src/app/`; do not implement an unapproved sign-in or invitation UX.
- [ ] T013 Document the external initial-admin prerequisite in `README.md` or the appropriate operations documentation. Do not invent an in-app initial-admin creation flow; coordinate the separately owned provisioning process before a family space is launched.

**Checkpoint**: Foundation is ready only when migrations, access rules, and authorization checks have been reviewed. This is a future implementation checkpoint, not a claim that they exist or pass.

## Phase 3: User Story 1 — Add and track family tasks (P1)

**Goal**: An admin can add a task to the one shared family list; family members can view it with its details.
**Independent verification**: With an authorized admin and an existing family space with eligible member(s), create a task with title, one assignee, and priority; confirm all members can view it and a non-admin cannot create it.

### Tests

- [ ] T014 [P] [US1] Add integration coverage in `tests/integration/tasks-create-and-list.test.ts` for valid admin task creation and shared-list visibility to family members, mapped to FR-001–FR-005 and FR-009.
- [ ] T015 [P] [US1] Add permission coverage in `supabase/tests/` for denied non-admin task creation and denied nonmember reads, mapped to FR-003 and FR-013.

### Implementation

- [ ] T016 [US1] Implement family-space-scoped task queries in `src/features/tasks/` and `src/app/(family)/`; default list retrieval must include active and Done tasks per FR-015.
- [ ] T017 [US1] Implement admin task creation in `src/features/tasks/` and the family-list UI in `src/app/(family)/`; require nonblank title, exactly one family-member assignee, and required priority before saving (FR-001, FR-004–FR-005, FR-010–FR-011).
- [ ] T018 [US1] Render each task's title, assignee name, High/Medium/Low priority, and completion state in `src/components/` and `src/features/tasks/`; keep priority distinct from the assignee identifier (FR-014).
- [ ] T019 [US1] Add understandable empty, invalid-input, loading, and permission-denied states in `src/features/tasks/` and `src/app/(family)/`.
- [ ] T020 [US1] Verify task details and completion state for non-removed tasks persist across leaving and revisiting the list; add integration coverage in `tests/integration/tasks-persistence.test.ts` (FR-009, SC-004).

## Phase 4: User Story 2 — Assign responsibility and focus on priority (P2)

**Goal**: Admins can update task details; each task retains exactly one responsible family member and an explicit required priority.
**Independent verification**: As an admin, change an existing task's assignee and priority and confirm the list shows the updated member name and selected priority; confirm members cannot edit those fields.

### Tests

- [ ] T021 [P] [US2] Add integration tests in `tests/integration/tasks-edit.test.ts` for admin updates to title, same-space assignee, and priority; include rejection of invalid and cross-space assignees (FR-004–FR-006, FR-011).
- [ ] T022 [P] [US2] Add database authorization tests in `supabase/tests/` proving a member cannot change title, assignee, or priority and that task priority remains a separate High/Medium/Low value (FR-005, FR-013–FR-014).

### Implementation

- [ ] T023 [US2] Implement admin-only task editing in `src/features/tasks/` and `src/app/(family)/`; preserve same-space assignee and required-priority constraints (FR-006, FR-011).
- [ ] T024 [US2] Add editable assignee and priority controls in `src/components/`; show the assignee by name and use any selected per-person color only as a supplement, never as the sole identifier or priority signal (FR-014).
- [ ] T025 [US2] Add task update success and validation/error states in `src/features/tasks/`; preserve existing data on rejected updates.
- [ ] T026 [US2] Verify priority focus outcome with a usability check that asks intended users to identify a High-priority task; record method and results against SC-003 and SC-005 without assuming a pass in advance.

## Phase 5: User Story 3 — Complete and review tasks (P3)

**Goal**: Any family member can mark a task Done or reopen it, and Done tasks remain visible and distinguishable.
**Independent verification**: A member marks an active task Done and later reopens it; each state persists on revisiting the shared list, and Done tasks appear in the default view.

### Tests

- [ ] T027 [P] [US3] Add integration tests in `tests/integration/tasks-completion.test.ts` for member completion, reopening, persistence, and visibility of Done tasks alongside active tasks (FR-008–FR-009, FR-015; SC-004, SC-008).
- [ ] T028 [P] [US3] Add authorization tests in `supabase/tests/` proving all members can change completion state while non-admins cannot edit or remove tasks (FR-008, FR-013).

### Implementation

- [ ] T029 [US3] Implement completion and reopen controls for family members in `src/features/tasks/` and `src/components/`, routed through the constrained completion operation from T009 (FR-008).
- [ ] T030 [US3] Make active and Done states visually distinguishable in `src/components/` and keep Done tasks in the default list in `src/features/tasks/` (FR-015).
- [ ] T031 [US3] Verify that completion changes persist and that reopening preserves title, assignee, and priority; record evidence against the specification's edge cases and SC-004.

## Phase 6: Admin membership and task removal

**Purpose**: Implement remaining functional requirements without inventing unresolved onboarding or retention behavior.

- [ ] T032 [US1] Before implementing member invite/add UX, resolve the deferred `member-invitation-flow` Question and confirm the account association/acceptance behavior; then implement the approved admin-only flow in `src/features/family/` and `src/app/(family)/` (FR-012). Keep this task blocked until that product decision is available.
- [ ] T033 [US1] Implement membership authorization so only current family-space members can read the shared list, and test removed/uninvited users cannot access it in `supabase/tests/` (FR-003, FR-012; SC-007).
- [ ] T034 [P] [US1] Before implementing task removal storage behavior, resolve the deferred `removed-task-retention` Question and any applicable security/retention policy; then implement admin-only removal in `src/features/tasks/` and `supabase/migrations/` consistent with the decision (FR-007, FR-013). Do not infer physical deletion, soft deletion, retention duration, or restoration capability.
- [ ] T035 [US1] Add tests in `tests/integration/tasks-removal.test.ts` and `supabase/tests/` proving only admins can remove tasks and removed tasks no longer appear in the shared list (FR-007, FR-013).
- [ ] T036 [US1] Before implementing sign-in UX, resolve the deferred `email-signin-flow` Question; then implement the chosen email authentication flow in `src/app/(auth)/` and `src/lib/supabase/` without changing the selected Supabase Auth direction.

## Phase 7: Polish and cross-cutting verification

- [ ] T037 [P] Add responsive and accessible labels, keyboard behavior, and status/priority announcements in `src/components/` and `src/app/`; verify against the constitution's inclusive-usability principle.
- [ ] T038 [P] Add usability evaluation tasks for task creation, shared-list comprehension, assignee/priority identification, and priority focus; record observed results against SC-001–SC-003 and SC-005 without pre-claiming thresholds.
- [ ] T039 Run the selected repository's typecheck, lint, unit, component, integration, and Supabase authorization suites; record actual commands, results, and any failures. Do not mark this task complete before execution.
- [ ] T040 Review `specs/001-todo-app/quickstart.md` against the implemented setup and verification commands; update it only through the documentation review workflow if it is stale.
- [ ] T041 Review the open document consistency findings CHK009 and CHK010; create new drafts for stale approval-status text in `spec.md`, `data-model.md`, and `quickstart.md` through their owning document workflows, preserving locked versions, then obtain owner review of those revisions.
- [ ] T042 Perform a final requirements traceability review for FR-001–FR-016 and SC-001–SC-008; record evidence and unresolved deviations before release review.

## Dependencies and Execution Order

- Setup tasks T001–T005 precede implementation. T001 confirms repository paths before relying on proposed locations.
- Foundation tasks T006–T013 depend on the selected repository/stack setup. RLS and authorization tests must be in place before exposing family/task data.
- User stories depend on foundation completion. Within a story, write relevant tests before or alongside implementation; establish failures before implementation where the selected test workflow supports test-first execution.
- US1 (T014–T020) is the P1 MVP and precedes dependent list editing and completion behavior.
- US2 (T021–T026) and US3 (T027–T031) depend on shared task/list foundations; they may be developed independently only after foundation and required shared components are stable.
- Membership onboarding (T032) is gated by the existing invitation-flow product question. Initial-admin provisioning remains an external operational prerequisite (T013), not an invented in-app workflow.
- Removal implementation (T034–T035) is gated by the existing removal/retention product question. Non-removed task persistence remains required regardless.
- Authentication UI (T036) is gated by the existing email sign-in product question; the owner selected Supabase email/auth but did not select the sign-in method.
- Polish and release verification T037–T042 follow implementation of the relevant stories and must be based on recorded evidence.

## Parallel opportunities

- After T001 confirms actual repository conventions, independent setup tasks T003–T005 may be parallelized where they do not edit the same configuration files.
- Foundation T007 and T011 can proceed independently of schema/RLS work when their shared interfaces are agreed; T010 depends on the policies and migration it tests.
- US2 and US3 test authoring may proceed independently after shared task and membership interfaces are stable.
- Accessibility/usability work in T037–T038 can proceed alongside story implementation when UI behavior is available, but final evaluation depends on a working build.
- Do not parallelize edits that contend on the same migration, shared task component, or authorization policy files.

## MVP and implementation strategy

1. Confirm repository structure and setup (T001–T005).
2. Complete and review the shared data/access foundation (T006–T013).
3. Deliver and independently verify US1 (T014–T020) as the initial MVP increment.
4. Add US2 and US3 in priority order, preserving separate story verification.
5. Resolve the product questions before implementing their dependent auth, invitation, and removal behaviors.
6. Complete cross-cutting verification, quickstart review, traceability, and document consistency follow-up (T037–T042).

All items above are planned work only. None is marked complete; no code has been run and no tests are claimed as executed.