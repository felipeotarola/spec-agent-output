# Quickstart: Family ToDo App

**Basis**: Approved `spec.md` v9 and constitution v1; submitted technical direction from Plan form `plan-technical-direction` (submission `05ff6061-5267-4bdd-8077-581dbf766f10`). Proposed architecture and structure are described in current `plan.md` v3, a draft awaiting owner review. This is a setup and verification guide, not evidence of implementation or executed checks.

## Prerequisites

- A selected source repository using a Node.js version compatible with its chosen Next.js release.
- The owner-selected web app / Next.js / React / TypeScript / Supabase database and email-auth direction.
- A Supabase project with Auth and PostgreSQL enabled, configured for the email sign-in method chosen during implementation. The precise flow and email delivery settings are not yet specified.
- Local environment configuration supplied securely by the project owner. Never commit credentials or place privileged keys in browser-accessible configuration.
- Exact package, runtime, hosting, email-auth flow and test-runner versions remain implementation choices; first reconcile this guide with the selected repository.

## Setup outline

1. Clone/open the selected repository and inspect its existing conventions before adding files. Preserve an established structure where it differs from the proposal in `plan.md`.
2. Install dependencies using the repository's lockfile and documented package-manager command. Add Next.js, React, TypeScript, and the appropriate Supabase SSR/client integration only if not already present.
3. Create or select a Supabase project. Configure Supabase Auth for the email-based login direction. The submitted product answer does not select password versus OTP/magic link, verification policy, recovery flow, or email delivery provider; do not assume these settings.
4. Configure local environment variables using the chosen Supabase integration. Use the exact variable names expected by installed packages. Keep any service-role/privileged credential server-only and use it only if a reviewed operation requires it; prefer user-session-scoped database access protected by RLS. Never commit credentials or expose privileged keys to client code.
5. Add versioned SQL migrations for family spaces, memberships, tasks, constraints, indexes, RLS policies, and any narrowly scoped operations. Follow the proposed invariants in `data-model.md`: only members read their family space, admins manage tasks, all members may change completion state only, assignees belong to the same space, and default list queries include Done tasks. Apply restrictive access policies before loading real family data.
6. Establish the external owner/support process that assigns the initial admin to a family task space, as required by FR-016. This provisioning workflow is outside the in-app feature behavior; secure and document operational ownership separately before launch.
7. Start the application using the repository's development command. Verify login, membership-scoped list access, task operations, and permission failures using non-production test accounts and data.

## Verification checklist for implementation

These are proposed checks, not completed test results:

- An admin can create a task with a nonblank title, exactly one in-space assignee, and required High/Medium/Low priority (FR-001, FR-004, FR-005, FR-010, FR-011).
- An admin can edit and remove tasks; a member cannot create, edit, or remove them (FR-006, FR-007, FR-013).
- Any family member can mark a task Done or reopen it, without changing its other fields (FR-008, FR-013; Edge Cases).
- All members of one family space can see its shared list; an unaffiliated user and a member of a different space cannot read it (FR-002, FR-003, FR-012).
- Active and Done tasks both appear in the default view (FR-015). Task details and completion state persist after revisiting for tasks that have not been removed under FR-007 (FR-009, SC-004).
- Removing a task excludes it from the shared list. Do not infer physical deletion, recovery, or a retention period; verify the chosen implementation against any separately approved retention policy.
- Assignee name and priority are separately visible; color is not the only assignee identifier (FR-005, FR-014).
- Empty or whitespace-only titles, missing assignee, and missing priority are rejected with understandable correction feedback (FR-010, FR-011).
- Database/RLS tests exercise cross-family isolation, same-space assignee integrity, and admin/member permissions; do not treat UI-only checks as authorization evidence.
- No check is considered passed until actually executed and its result recorded by the implementation team.

## Not yet specified

The repository URL and conventions, deployment/hosting, exact auth UX and email configuration, invitation acceptance, initial-admin operations, retention/deletion, browser support, numerical performance targets, and test framework remain open. Resolve relevant items before production deployment; do not store credentials in this document. The task-removal question remains an implementation-stage decision; no policy is implied here.
