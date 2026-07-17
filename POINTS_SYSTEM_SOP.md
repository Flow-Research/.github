# Flow Research Points System — Standard Operating Protocol

## 1. Purpose

This SOP governs the collection, scoring, storage, correction, publication, and operation of the Flow Research points system. It applies to GitHub ingestion, Discord submissions and reviews, manual awards, leaderboards, public APIs, scheduled jobs, migrations, and future integrations.

The objective is a system that is accurate, repeatable, auditable, private by default, easy to operate, and resistant to duplicates or silent corruption.

## 2. System invariants

These rules must always hold:

1. One real contribution produces at most one effective award.
2. Re-running an ingestion or review operation does not change the final state.
3. Every award has a stable identity, source, actor, timestamp, status, and scoring basis.
4. Conflicting awards are never resolved silently.
5. Public outputs never expose private repository links or internal metadata.
6. Disabled repositories and unsupported event types are not ingested.
7. A failed required scan cannot be reported as a successful complete run.
8. Corrections remain auditable.
9. Scoring is deterministic under a declared rule version.
10. Production writes can be previewed before they are applied.

## 3. Sources of truth

- `GitHub_Repos`: repository tracking registry and workstream mapping.
- `Contributions`: authoritative award ledger.
- Shared normalization module: repository aliases and canonical identity rules.
- Versioned scoring policy: point values and classification rules.
- Public-output sanitizer: the only approved path from internal records to public surfaces.

Configuration must not be duplicated across environment variables, scripts, and sheets unless one is explicitly documented as a temporary override.

## 4. Canonical identity

### GitHub pull requests

Canonical format:

`github-pr:<normalized-owner>/<normalized-repository>:<pull-request-number>`

Normalization must:

- lowercase owner and repository for identity comparison;
- map historical or renamed repositories to their canonical repository;
- recognize legacy event IDs and evidence URLs;
- reject malformed owner, repository, or PR numbers.

### Discord submissions and reviews

Use stable Discord message/submission/review identifiers, not titles or usernames. Usernames may change and must not be identities.

### Manual awards

Manual awards require a generated immutable event ID and an explicit dedupe key based on the underlying event or approved administrative decision.

## 5. Repository onboarding and retirement

### Onboarding checklist

Before setting `ingest_enabled = TRUE`:

- confirm the exact repository identity;
- assign a default workstream;
- record repository visibility;
- decide whether historical backfill is allowed;
- identify bot/dependency PR policy;
- test access with a dry run;
- verify that private evidence is suppressed from public outputs.

### Retirement checklist

- set `ingest_enabled = FALSE`;
- preserve historical contribution records;
- add alias mapping if the repository was renamed or replaced;
- do not delete history merely because a repository is deprecated;
- verify scheduled jobs no longer scan it.

## 6. Normal ingestion procedure

1. Acquire a run ID and record mode: dry run or apply.
2. Load the repository registry.
3. Select only enabled repositories.
4. Resolve aliases to canonical identities.
5. Scan all required repositories.
6. Stop the apply run if any required scan fails.
7. Build canonical identities for discovered events.
8. Compare against event ID, dedupe key, legacy identity, and current batch.
9. Classify and score using the active scoring-rule version.
10. Place ambiguity or point conflicts into manual review.
11. Produce a summary before writing.
12. Apply only with an explicit flag.
13. Re-read persisted data and reconcile counts.
14. Record run metrics and failures.

## 7. Required run summary

Every sync must report:

- run ID;
- mode;
- repositories requested, scanned, skipped, inaccessible, and failed;
- events discovered;
- existing events;
- proposed awards;
- conflicts;
- rejected records;
- records written;
- duration;
- scoring-rule version.

An unexpected zero scan, large volume change, or access failure must be visible and alertable.

## 8. Scoring QA

Scoring logic must be isolated from GitHub transport and sheet-writing code.

Required tests:

- each scoring category and boundary;
- same input produces same score;
- unknown input does not receive an invented score;
- bot and dependency PR policy;
- label-based override policy;
- historical scoring fixtures;
- scoring-rule version recorded with output.

A scoring-policy change must state:

- why it changed;
- affected contribution types;
- effective date;
- whether historical awards remain unchanged or are migrated;
- migration and rollback method.

## 9. Duplicate and conflict handling

### Proven duplicate

A row may be removed or reversed only when canonical identity, contributor, source event, and intended award are proven equivalent.

### Point conflict

When two rows represent the same contribution but have different points:

- stop automatic cleanup;
- show both records and evidence;
- obtain an explicit policy decision;
- retain the chosen award;
- canonicalize its identity;
- remove or reverse the rejected duplicate;
- record the rationale.

### Routine duplicate audit

Run a duplicate audit after:

- backfills;
- alias changes;
- migrations;
- scoring rewrites;
- incident repair;
- large imports.

The audit must compare canonical event identity, dedupe key, and normalized evidence identity.

## 10. Corrections and data repair

Before any destructive repair:

1. stop scheduled writers and announcements;
2. create and verify a backup;
3. identify exact rows and expected totals;
4. run a dry-run repair;
5. patch the root cause first;
6. apply deterministic changes;
7. verify row count, identities, and contributor totals;
8. verify no privacy regression;
9. restart services gradually;
10. document the incident and prevention.

Prefer reversal-plus-replacement or explicit adjustment records when audit history matters. Direct deletion is reserved for proven redundant duplicates with a backup and recorded rationale.

## 11. Public-output protocol

All public data must pass through one shared sanitizer.

Public allowlist may include only approved fields such as display name, approved points total, category, workstream, public title, and verified-safe public evidence.

Never publish:

- private repository URL or name;
- internal notes;
- automation run IDs;
- reviewer/admin identifiers unless intentionally public;
- raw Discord IDs;
- tokens, secrets, or authorization data;
- unapproved or disputed awards.

Discord announcements and public API responses must use the same evidence-safety rule.

## 12. Service operation

### Long-running services

- API and Discord bot run under a process manager.
- Process definitions belong in a version-controlled ecosystem configuration.
- Restart commands, environment requirements, health checks, and log locations must be documented.

### Scheduled sync

- Scheduled sync uses explicit `--apply`.
- Manual invocation defaults to dry run.
- Prevent overlapping runs with a lock.
- On failure, exit non-zero and do not send success notifications.
- A recurring job must not be enabled until a dry run and one supervised apply run pass.

### Startup verification

After start or restart:

- process status healthy;
- no crash loop;
- expected repository count loaded;
- sheet/API access works;
- public privacy test passes;
- no duplicate notifications emitted.

## 13. Backup and recovery

- Create regular versioned backups of the authoritative ledger.
- Create an additional backup before bulk changes.
- Recovery procedure must be tested, not merely documented.
- Backups must not be publicly shared.
- Record backup ID, creation time, source revision, and reason.

## 14. Change-control requirements

Every pull request that changes points behavior must include:

- problem and impact;
- invariants affected;
- data/schema effects;
- idempotency and dedupe behavior;
- privacy review;
- tests added;
- dry-run evidence;
- migration and rollback;
- operational changes;
- post-deploy checks.

High-risk changes require human review even when generated by an agent.

## 15. Recurring QA schedule

### Every sync

- validate repository load and scan completeness;
- report counts and conflicts;
- confirm no duplicate writes.

### Weekly

- duplicate audit;
- failed-run review;
- enabled-repository/access reconciliation;
- public-output privacy smoke test;
- check unresolved scoring conflicts.

### Monthly

- restore-test latest backup;
- review repository registry for obsolete or missing projects;
- inspect scoring anomalies and bot awards;
- review permissions and secrets;
- prune dead code and stale operational documentation.

### Before major release

- full dry-run reconciliation;
- migration rehearsal;
- rollback rehearsal;
- load/rate-limit review;
- privacy and security review;
- agent-instruction review.

## 16. Definition of healthy

The points system is healthy when:

- all intended repositories are enabled and accessible;
- repeated syncs produce no new records without new source events;
- duplicate audit returns zero unresolved ordinary duplicates;
- every remaining conflict is explicitly tracked;
- public outputs contain no private evidence;
- backups are current and restorable;
- scheduled jobs are observable and non-overlapping;
- scoring rules and operations are documented and tested.