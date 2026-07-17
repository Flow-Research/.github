# Flow Research Coding Agent Instructions

These instructions apply to every coding agent working in a Flow Research repository. Repository-local `AGENTS.md` files may add stricter rules but must not weaken these requirements.

## Mandatory operating mode

1. Read the repository README, contribution guide, local `AGENTS.md`, tests, package scripts, and deployment configuration before editing.
2. State the problem, invariants, non-goals, and expected outcome before proposing a large change.
3. Inspect all callers and outputs of code being changed.
4. Prefer the smallest coherent patch.
5. Do not perform destructive production actions without explicit authorization, backup, dry run, and exact affected-record preview.
6. Do not claim success without running applicable checks and reporting their actual output.

## Engineering requirements

### Correctness

- Validate at boundaries.
- Handle expected failure paths explicitly.
- Do not invent missing business values or silently downgrade errors.
- Preserve backward compatibility unless the change explicitly includes a migration.

### Idempotency and duplicates

- Every write path must define a stable identity or idempotency key.
- Re-running the operation must not create duplicate records, side effects, notifications, or awards.
- Check existing persisted data and duplicates created within the same batch.
- Add a repeat-run test for ingestion, migration, webhook, review, and scheduled-job changes.

### Simplicity

- Do not add abstractions, services, queues, caches, databases, dependencies, or frameworks without a demonstrated current need.
- Avoid speculative architecture.
- Do not mix broad refactoring with a behavior change unless necessary for safety.
- Reuse established modules and patterns before creating new ones.

### Clear and modular code

- Separate domain rules from I/O, transport, storage, and presentation.
- Keep functions focused and names explicit.
- Centralize shared normalization, validation, privacy, and identity logic.
- Avoid hidden global state and surprising environment-dependent behavior.
- Comments explain intent, tradeoffs, or invariants rather than restating code.

### Privacy and security

- Treat repository visibility and data sensitivity as unknown until verified.
- Never expose private repository links, secrets, internal identifiers, raw tokens, or unnecessary personal data.
- Public outputs use explicit allowlists.
- Do not print secrets or private evidence in logs, tests, examples, diffs, or summaries.
- Use least privilege and preserve existing security boundaries.

## Required verification

Run all applicable repository checks, including:

- syntax or compilation;
- formatting and linting;
- type checks;
- unit and integration tests;
- duplicate/retry test;
- malformed-input test;
- partial-failure test;
- privacy-output test;
- migration dry run and reconciliation when data changes.

When a check cannot run, explain exactly why and what remains unverified.

## Data and migration rules

- Back up before destructive or bulk changes.
- Administrative scripts default to dry run and require an explicit apply flag.
- Refuse to apply when required inputs or targets fail to load.
- Emit run IDs and before/after counts.
- Preserve audit history for corrections and overrides.
- Never silently resolve conflicting records.

## Pull request expectations

Every substantive PR should describe:

- problem and scope;
- invariants and risks;
- implementation approach;
- idempotency and dedupe behavior;
- privacy/security impact;
- tests and evidence;
- migration and rollback;
- operational or configuration changes.

## Points-system changes

For points, awards, repository ingestion, Discord review, public leaderboard, or sync code, also follow `POINTS_SYSTEM_SOP.md` and `ENGINEERING_QUALITY_PROTOCOL.md` in the Flow Research `.github` repository.

Key non-negotiable invariants:

- one contribution has at most one effective award;
- retries do not change final state;
- conflicts require explicit review;
- public outputs never expose private evidence;
- a failed required scan cannot be treated as a complete successful run;
- scoring is deterministic and versioned.

## Stop conditions

Stop and ask for explicit direction when:

- two records represent the same event but disagree materially;
- a private/public classification is uncertain;
- a destructive action lacks a verified backup;
- requirements conflict with an invariant;
- production configuration differs materially from repository assumptions;
- the requested approach creates avoidable security, privacy, or data-integrity risk.