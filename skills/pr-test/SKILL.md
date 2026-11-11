---
name: pr-test
description: "Use when testing or verifying a pull request end-to-end — exercise it as a regular user (UI + database), run its migrations/seeders/operations/scripts, verify linked tickets, and prove the system still works past the diff. Reports findings in chat; leaves production, the PR, and the repo untouched; typically post-approval."
---

## Required `<project-root>/docs` reads

Shell-read before testing the PR; may be gitignored — missing → note gap, never invent: `<project-root>/docs/code-navigation.md`, `<project-root>/docs/database-tools.md`, `<project-root>/docs/git-hosts.md`; `<project-root>/docs/issue-trackers.md` (only when the PR references tickets/cards), `<project-root>/docs/browser-automation.md` (Step 7 only)

Test whether a PR **works**, not whether it reads well — and whether the **system** still works around it. The diff marks where the change landed; the blast radius is every behavior, caller, and data artifact it can break. A green feature check inside a broken system is a failed test.

**Boundary:** the local environment is fully mutable — checkouts, migrations, seeders, operations, browser-driven writes, and local DB writes all execute as part of testing, without user confirmation. Untouched scope: production systems, the PR thread, and the repo's files and history.

### 1. Interpret Arguments

PR number/URL → `<pr-ref>`; extra focus areas or constraints → `<additional-context>`; no arguments → derive from conversation or the open PR for the current branch (git host list PRs, `git-hosts.md`).

### 2. Load PR Context

View PR + diff via git host (`git-hosts.md`). Extract `<pr-purpose>` (why — title, body, commit messages), `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`, `<changes>`, `<ticket-refs>` (every issue/card reference in body, commits, or branch name — git host or external tracker). Unloadable → STOP.

Classify `<changes>`. Flag **data artifacts** — migrations, seeders, operations, one-off scripts — code whose only real test is execution against a populated database. Flag shared/infra files (config, middleware, providers, base classes, factories) — blast radius spans the whole app. Both feed the test plan; neither is covered by a UI scenario alone.

### 3. Load Linked Context

For each `<ticket-refs>` entry, load via the issue tracker load tool (`issue-trackers.md`): body, every comment, checklists, attachments; traverse nested links per the spec — every nested ticket feeds `<expected-outcomes>`. Every ticket, spec, and card load — top-level and nested — goes through the tracker tooling documented there alone; the browser is reserved for exercising the app in Step 7. A ref the tracker tooling cannot load → gap per below, no browser fallback.

Extract `<expected-outcomes>` — one entry per acceptance criterion, validation item, or spec requirement the PR must deliver. A ticket that fails to load → record a gap, don't guess its criteria.

Classify `<request-source>` per ticket: **client-origin** — filed in the customer-facing tracker or carrying a client requester/thread (client bug reports, client-specific feature requests) — or **internal**. Client-origin → extract `<client-cases>`: the reported steps, the records/data and module involved, and the symptom or use case in the client's own terms.

Build `<coverage-map>`: each `<expected-outcomes>` entry → the diff files/modules that should deliver it. Flag outcomes with no corresponding change — the PR may not address every linked card.

**Done when:** every linked ticket loaded or gapped · every criterion/validation item in `<expected-outcomes>` · `<request-source>` classified per ticket, `<client-cases>` extracted where client-origin · coverage map built with gaps flagged.

### 4. Align Local Branch

Record `<previous-branch>` via `git branch --show-current`. Uncommitted changes → STOP, ask user to stash/commit. Check out via git host checkout PR or CLI fallback (`git-hosts.md`); fails → STOP. Restore `<previous-branch>` when testing ends.

### 5. Build the Test Plan

From `<coverage-map>`, write `<test-plan>` with four scenario classes:

- **Source** — one scenario per `<client-cases>` entry, replaying the client's case as reported: their steps, their data shapes, their flow. A client bug report → the exact reported failure must now pass; a client-specific feature request → the client's use case performed the way they described it. These are the primary scenarios — Delivery generalizes from them, never substitutes; an `<expected-outcomes>` entry a source scenario already covers needs no duplicate.
- **Delivery** — one user-level scenario per `<expected-outcomes>` entry: entry point, user action, expected observable result (UI state, DB effect, or both).
- **Data-artifact** — one execution scenario per migration, seeder, operation, or script flagged in Step 2: run against the populated local dev DB; assert schema lands and existing rows survive (migrations); expected rows/effects land (seeders, operations, scripts); re-run safe where idempotency is claimed.
- **Regression** — the surface past the diff: direct callers and shared modules traced via code navigation (`code-navigation.md`), each asserting unchanged behavior; project tests covering the changed symbols, run by file/name filter; and a **system pass**: the core journeys every user hits (auth, landing view, primary CRUD flow) exercised even where the diff never touches them.

Order setup-free flows first; note test-data prerequisites per scenario.

**Done when:** every `<client-cases>` entry maps to a source scenario · every `<expected-outcomes>` entry maps to a delivery scenario · every flagged artifact maps to an execution scenario · regression callers enumerated · system-pass journeys picked.

### 6. Provision Test Data

Before exercising, verify the local DB holds the records each scenario needs — the client case's data shapes, tenant context, required relationships. Missing → provision it: seeds/factories for generic data; production copies when a scenario reproduces a production case or needs real data shapes.

Production copy rules:

- Follow project database-tool and data-handling rules (`database-tools.md`). Confirm the source is production and the destination is local before importing.
- Copy only the records needed for the scenario, including required relationships and tenant context. Preserve reproduction-critical values; mask unrelated sensitive data and exclude secrets from logs, reports, and version control.
- Isolate local mail, queues, webhooks, and external integrations before exercising copied data so reproduction cannot affect real users or services.
- Write only to local — never to production. Ask before overwriting or deleting existing local data.

**Done when:** every scenario's required records exist locally — seeded, copied, or already present.

### 7. Exercise the Change

Run the app per project tooling. Drive each scenario through browser automation (`browser-automation.md`) as a **regular user** — real UI navigation, forms, clicks. No console shortcuts, direct API calls, or DB writes to force state — Step 6 provisions whatever data a scenario requires.

After each scenario, verify **persisted effects** via read-only DB inspection (`database-tools.md`): rows created/updated, status transitions, relationship integrity. Live DB is ground truth — a UI that looks right while the DB is wrong is a failure.

Execute each data-artifact scenario per project tooling against the populated local dev database — the same DB the browser scenarios exercise, so artifact effects are visible to UI checks — then inspect live schema and rows. A migration that applies but strands stale columns, or a seeder that runs clean but writes wrong rows, is a failure. Artifact output text is not evidence; the database is.

Regressions: run diff-related tests and lint where they exist, plus every regression scenario and the system pass. A test failure the diff did not cause is still a finding — mark it pre-existing after a cheap check against `<base-branch>`.

Record `<results>` per scenario: `pass`/`fail`/`blocked` with evidence (observed UI state, query results, error text).

**Done when:** every scenario attempted · each `<expected-outcomes>` entry has a result · every artifact executed or `blocked` with reason · related-test results and system pass captured.

### 8. Report Findings

Chat only — never post to the PR. Per `<expected-outcomes>` entry: verified / failed / untested (with reason). Then:

- **Issues** — severity, reproduction steps, evidence, `file:line` where the diff is implicated.
- **Coverage gaps** — linked tickets/cards the PR does not fully address (`<coverage-map>` flags + failed scenarios).
- **Data artifacts** — what each executed, or why `blocked`.
- **Regressions** — adjacent behavior that broke: regression scenarios, test failures (pre-existing vs caused), system-pass failures.
- **Verdict** — `works` only when delivered outcomes pass AND the system still works around them; otherwise `issues found`, one line.

## Notes

- Verifies, never fixes — issues found → user invokes a fix workflow; leave the repo as found (no edits, no stray commits).
- A scenario that cannot be exercised locally (external-service dependency, data not provisionable from seeds, factories, or production copies) → `blocked` with reason — never `pass` unverified.
- An artifact too destructive to run locally (drops tables, mutates shared infra) → `blocked` with the risk named; review its code and report what verification the deploy needs instead.
