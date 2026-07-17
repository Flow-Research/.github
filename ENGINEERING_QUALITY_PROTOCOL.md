# Flow Research Engineering Quality Protocol

## Scope

This protocol applies to every Flow Research repository, service, script, data workflow, CI job, coding agent, and production operation. It is the default standard unless a repository documents a stricter one.

## Principles

1. Correctness before speed.
2. One authoritative source of truth per concern.
3. Idempotency by default: repeating an operation must not duplicate effects.
4. Canonical identity for every external object.
5. Safe failure: partial failure must not silently corrupt state.
6. Privacy by design: private links, secrets, and internal metadata stay private.
7. Small, reversible changes with dry runs and rollback paths.
8. Simple before clever; no speculative infrastructure.
9. Clear ownership, auditability, and observability.
10. Tests protect business invariants, not merely code coverage.

## Required engineering rules

### Identity and duplicates

- Define a deterministic canonical key before building ingestion.
- Normalize case, renamed repositories, aliases, and equivalent URL forms.
- Dedupe against persisted records and the current in-memory batch.
- Never use mutable titles, display names, or timestamps as the sole identity.
- Conflicting duplicates require manual review; do not silently merge them.
- Add uniqueness constraints where the storage system supports them.

### Idempotency

Every write path must answer: “What happens when this runs twice?”

Required controls:

- deterministic idempotency key;
- read-before-write or atomic upsert;
- same-batch duplicate protection;
- retry-safe external calls;
- duplicate-notification prevention;
- a test that runs the operation twice and proves the final state is unchanged.

### Data integrity

- Validate input at every system boundary.
- Reject malformed, incomplete, out-of-scope, or unexpected records before writing.
- Back up data before destructive cleanup or migration.
- Destructive tools default to preview/dry run and require explicit apply.
- Record actor, reason, time, run ID, previous value, and new value for corrections.
- Do not infer financially or reputationally meaningful values when evidence is missing.

### Privacy and secrets

- Public serializers use explicit allowlists, never broad object spreading.
- Private repository names and links must not appear in public APIs, leaderboards, Discord messages, logs, screenshots, or exports.
- Repository visibility is unknown until verified from an authoritative source.
- Secrets belong in environment or secret-management systems, never source code or sheets.
- Logs must not contain tokens, authorization headers, or unnecessary personal data.

### Simplicity and modularity

- Use the least complex design that safely solves the present problem.
- Do not add a queue, cache, service, framework, or datastore without a concrete need and removal path.
- Separate business rules from transport, storage, and presentation.
- Put shared identity, normalization, validation, and privacy rules in reusable modules.
- Functions should have one responsibility and names should express intent.
- Comments explain why; code should explain what.
- Avoid hidden global state and implicit environment-dependent behavior.
- Delete dead code instead of preserving indefinite alternatives.

## Standard delivery process

### 1. Define

Document the problem, users, invariants, non-goals, privacy classification, canonical identities, success/failure behavior, and rollback.

### 2. Inspect

Read current code and tests, identify callers and outputs, inspect real data shapes and aliases, confirm production configuration, and identify scheduled/concurrent writers.

### 3. Design

Specify source of truth, read/write boundaries, idempotency, dedupe, partial-failure behavior, privacy controls, audit records, migration, and rollback.

### 4. Implement

Make the smallest coherent patch. Keep destructive behavior disabled by default. Refuse to continue when required inputs or targets fail. Do not mix broad refactoring with behavior changes.

### 5. Verify

Run syntax/type/lint checks and test:

- happy path;
- duplicate/retry path;
- malformed input;
- partial failure;
- privacy output;
- dry-run counts;
- before/after reconciliation.

### 6. Release

Use a reviewed pull request, record configuration changes, back up before migrations, deploy one controlled change at a time, and verify health and invariants immediately.

### 7. Observe

Every production automation should emit run ID, mode, targets, start/end time, and counts for scanned, skipped, created, updated, conflicted, and failed records.

Alert on invariant violations, repeated failures, unexpected zero scans, abnormal volume, and privacy-risk conditions.

## QA gates

A change is not complete until applicable gates pass:

- **Scope:** problem and non-goals are clear; no unrelated refactor.
- **Identity:** canonical keys and aliases are defined; retries and duplicates tested.
- **Integrity:** invalid input rejected; partial failure is safe; destructive work has backup and preview.
- **Privacy:** public outputs are allowlisted; private links and secrets are absent.
- **Maintainability:** code is clear; shared rules are modular without premature abstraction.
- **Release readiness:** checks pass; dry-run/staging evidence exists; rollback and post-deploy checks are defined.

## Incident protocol

When duplicates, corruption, privacy leakage, or incorrect scoring are suspected:

1. Stop scheduled writers and notifications.
2. Preserve logs and take a backup.
3. Identify the exact affected identity set.
4. Reproduce in dry-run mode.
5. Patch the root cause before repairing data.
6. Repair with deterministic, reviewable operations.
7. Reconcile totals and prove no remaining duplicates.
8. Restore services gradually.
9. Record cause, impact, fix, and prevention.

## Definition of done

Code, tests, documentation, configuration, migration, privacy review, observability, rollback, and operational ownership are complete.