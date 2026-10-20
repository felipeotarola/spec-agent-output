# Implementation Plan: Family ToDo App

**Branch**: `001-todo-app` | **Date**: 2026-10-20 | **Spec**: [spec.md](./spec.md)

**Input**: Approved feature specification `specs/001-todo-app/spec.md` v9 and approved constitution `.specify/memory/constitution.md` v1. Technical direction is grounded in submitted Plan form `plan-technical-direction`, submission `05ff6061-5267-4bdd-8077-581dbf766f10`.

## Summary

Deliver a web application for one shared family task list. Admins can create, edit, and remove tasks and invite/add family members; all family members can view tasks and change completion state. Tasks require exactly one responsible family member and a High/Medium/Low priority. Done tasks appear alongside active tasks by default. Per approved FR-009 and SC-004, details and completion state persist for tasks that have not been intentionally removed under FR-007. Removal means a task is no longer part of the shared list; the specification does not decide whether storage uses physical deletion, soft deletion, or a retention mechanism.

The owner selected a web app, Next.js, React, TypeScript, Supabase database, and Supabase email-based authentication/login. Proposed implementation: a TypeScript Next.js application with server-side Supabase integration, Supabase Auth, and PostgreSQL; PostgreSQL row-level security (RLS) is the authoritative access boundary, with a narrowly scoped completion operation so non-admin members cannot change other task fields. These are technical plan proposals, not implemented behavior or further owner-approved product requirements.

## Technical Context

**Language/Version**: TypeScript; exact Node.js and package versions to be selected to match the repository and supported Next.js/Supabase releases at implementation.

**Primary Dependencies**: Next.js with React; Supabase JS/SSR integration and Supabase Auth. Exact package versions remain an implementation-time compatibility choice.

**Storage**: Supabase-hosted PostgreSQL for family spaces, memberships, and tasks; Supabase Auth for identity and email-based login.

**Testing**: TypeScript/Next.js unit and component tests plus integration/authorization checks against a Supabase-compatible test environment. Exact test runner is not selected; choose one compatible with the repository before implementation. Include RLS/permission tests for admin and member roles, cross-family isolation, persistence of non-removed tasks, and removal behavior without treating removal as a data-retention decision.

**Target Platform**: Web application, responsive for common mobile and desktop browsers. Specific browser support targets are not provided.

**Project Type**: Full-stack web application (Next.js frontend and server-side operations, Supabase managed authentication/database).

**Performance Goals**: No numerical latency or load target was provided. Keep the first version proportionate to a family-scale shared list; do not claim a measured target without evidence.

**Constraints**: Use the owner-selected Next.js, React, TypeScript, Supabase database, and Supabase email/auth login direction. Keep service-role credentials server-only. Treat family/task information as personal data. Enforce family-space access and task action permissions in the database, not solely in client UI. Exact email sign-in method, invitation delivery/acceptance flow, hosting provider, retention policy, and browser targets are unspecified.

**Scale/Scope**: One shared list per family task space in v1, with membership, admin/member permissions, tasks, required single assignee and High/Medium/Low priority, and active/Done task display. Personal/member-specific lists are excluded. No scale estimate was supplied.

## Constitution Check

| Principle / constraint | Plan response | Status |
|---|---|---|
| Family-centered usefulness | One shared list and membership-bound visibility serve family coordination (FR-002–FR-003, FR-012). | Pass |
| Simplicity first | Use one Next.js application and Supabase-managed auth/database rather than introducing separate services without need. | Pass |
| Priority and focus | Store explicit required High/Medium/Low priority and present it separately from assignee identity (FR-005, FR-014). | Pass |
| Usable and inclusive experience | Plan responsive web UI, clear textual labels, and accessible status/priority cues; detailed usability and accessibility checks belong to design and verification. | Pass; implementation verification pending |
| Quality through verification | Plan unit/component, integration, persistence, and role-based RLS checks mapped to requirements. | Pass; test execution pending |
| Personal-data care | Use authenticated identity, database-enforced membership boundaries, least privilege, and server-only privileged credentials. Retention/legal requirements remain unspecified. | Pass with open operational decisions |

No constitution violation is proposed. Re-check this gate after detailed design and before implementation. This plan does not establish that security or accessibility tests have been executed.

## Proposed Architecture and Decisions

1. **Application shape**: A single Next.js TypeScript application, using the App Router unless the selected repository already establishes another supported pattern. Use server-side data access/actions or route handlers for privileged operations; limit client components to interactive UI.
2. **Identity and persistence**: Supabase Auth provides email-based authentication; Supabase PostgreSQL stores app records. Keep the authenticated user context on Supabase queries so RLS evaluates the actual member. The precise email sign-in experience is not decided.
3. **Authorization**: Represent family-space membership and admin role in application data. Enable RLS on all family/task tables. Members may read tasks only in spaces where they are members; admins may create/edit/remove tasks; all members may change completion state only. Avoid shipping a service-role key to the browser. A narrowly scoped database function or equivalent server-side operation should enforce completion-only changes for members; confirm the final mechanism in implementation review.
4. **Task lifecycle and persistence**: Keep non-removed tasks and their title, assignee, priority, and completion state available when users leave and revisit the shared list (FR-009, SC-004). Admin removal excludes a task from the shared list (FR-007); it does not by itself specify whether the row is physically deleted, soft-deleted, or retained under a policy. Do not infer a retention period or promise recoverability from this requirement. Reconcile the implementation with any separately approved retention/deletion policy.
5. **Initial admin**: The specification places initial-admin assignment in an external owner/support process (FR-016). The application needs an initial admin membership before a family space is administered, but this plan does not invent the external provisioning workflow. Coordinate secure provisioning and operational ownership before launch.
6. **Member access and invitations**: Admins invite or add members (FR-012); only people added to the family task space may access its shared list (FR-003, FR-012). Supabase Auth is selected for email login, but concrete invitation UX, email delivery/configuration, and acceptance behavior are not specified. Do not assume a particular password, OTP, magic-link, or invitation-acceptance flow.
7. **List behavior and assignee presentation**: Fetch tasks scoped to the active family task space and show both active and Done tasks by default (FR-015). Display the responsible person's name; the specification includes a delegated proposal for a consistent per-person color as a supplement, never the sole identity and never a substitute for the distinct priority label (FR-014). No ordering/filtering behavior beyond the default Done visibility is specified.
8. **Error and empty states**: Validate nonblank title, exactly one eligible assignee, and required priority before submission; return understandable correction messages (FR-010–FR-011). Handle unavailable auth/session, permission denial, and empty task/member lists without exposing another family's data.

### Alternatives considered

- **Separate frontend and backend services**: Not selected for v1; Next.js server-side capabilities plus Supabase avoid extra deployable components without demonstrated need.
- **Client-only authorization**: Rejected because UI checks alone cannot protect family data or enforce member/admin distinctions. RLS and server-side identity are required.
- **Custom authentication/database service**: Not selected because the owner chose Supabase email/auth and database.
- **Specific email sign-in UX (password vs OTP/magic link)**: Not selected; the submitted answer identifies email auth/login but not the sign-in mechanism. Keep the choice open until auth UX is designed.
- **Physical vs logical task removal**: Not selected by the product spec. It defines removal from the shared list and persistence of non-removed tasks, not storage or retention semantics.

## Project Structure

### Documentation (this feature)

```text
specs/001-todo-app/
├── plan.md
├── data-model.md
├── quickstart.md
├── spec.md
└── checklists/
    └── requirements.md
```

### Proposed source layout

```text
src/
├── app/                         # Next.js routes, layouts, loading and error states
│   ├── (auth)/                  # Sign-in and auth callback routes
│   └── (family)/                # Authenticated family-list routes
├── components/                  # Shared UI components
├── features/
│   ├── family/                  # Membership and family-space UI/data access
│   └── tasks/                   # Task list, create/edit, completion flows
├── lib/
│   ├── supabase/                # Browser/server Supabase clients and session helpers
│   └── validation/              # Shared input validation
└── types/                       # Generated/shared database and domain types
supabase/
├── migrations/                  # Schema, constraints, RLS and database functions
└── tests/                       # Database/RLS authorization tests
tests/
├── unit/
├── components/
└── integration/
```

**Structure Decision**: Proposed single Next.js TypeScript app with Supabase SQL migrations and database tests. The actual repository has not been inspected or selected in the workspace; reconcile these proposed paths with the selected repository before implementation. Remove or adjust directories that repository conventions do not support.

## Supporting Artifacts

- `data-model.md` is applicable because family membership, task ownership, authorization, and the clarified task lifecycle depend on explicit relational constraints.
- `quickstart.md` is applicable as a proposed setup and verification guide for the selected stack; it contains no claims of executed tests.
- Separate `research.md` is not required: no external research was requested or performed, and the selected technologies come directly from the owner-submitted technical direction.
- Separate API contract files are not created at this stage: the feature has no separately specified public HTTP API. Database authorization and operation boundaries are described in the data model; revise this if implementation adopts an external API contract.

## Risks and Open Technical Decisions

- Email authentication is chosen, but password versus OTP/magic link, verification policy, recovery flow, and email provider configuration remain open.
- Invitation/acceptance behavior must align with admin-managed membership; exact flow and delivery are open.
- Initial-admin provisioning is a launch dependency owned by an external process; secure setup and operational ownership need definition before deployment.
- The storage and retention treatment of tasks removed under FR-007 is unspecified beyond removal from the shared list. Resolve any retention/security policy separately without weakening FR-009 for non-removed tasks.
- Exact hosting/deployment provider, environment management, browser support, and quantitative performance objectives were not supplied.
- RLS policy mistakes could disclose or corrupt family data; require automated tests for cross-family isolation and role boundaries before release.
- Repository structure and package/test versions must be confirmed against the actual selected source repository before implementation.
