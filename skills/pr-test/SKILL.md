---
name: pr-test
description: "Use when testing or verifying a pull request end-to-end — exercise the change as a regular user (UI + database), execute the migrations, seeders, operations, and scripts it ships, verify every linked ticket/card is delivered, and prove the whole system still works past the diff. READ-ONLY; reports findings in chat. Often run after the PR is approved."
---

## Required `<project-root>/docs` reads

Read via shell before testing the PR — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/browser-automation.md`
- `<project-root>/docs/code-navigation.md`
- `<project-root>/docs/database-tools.md`
- `<project-root>/docs/git-hosts.md`
- `<project-root>/docs/issue-trackers.md`

Test whether a PR **works**, not whether it reads well — and whether the **system** still works around it. The diff marks where the change landed; the blast radius is every behavior, caller, and data artifact it can break. A green feature check inside a broken system is a failed test.

### 1. Interpret Arguments

PR number/URL → `<pr-ref>`; extra focus areas or constraints → `<additional-context>`; no arguments → derive from conversation or the open PR for the current branch (git host list PRs, `<project-root>/docs/git-hosts.md`).

### 2. Load PR Context

View PR + diff via git host (`<project-root>/docs/git-hosts.md`). Extract `<pr-purpose>` (why — title, body, commit messages), `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`, `<changes>`, `<ticket-refs>` (every issue/card reference in body, commits, or branch name — git host or external tracker). Unloadable → STOP.

Classify `<changes>`. Flag **data artifacts** — migrations, seeders, operations, one-off scripts — code whose only real test is execution against a populated database. Flag shared/infra files (config, middleware, providers, base classes, factories) — blast radius spans the whole app. Both feed the test plan; neither is covered by a UI scenario alone.

### 3. Load Linked Context

For each `<ticket-refs>` entry, load via the issue tracker tool (`<project-root>/docs/issue-trackers.md`): body, every comment, checklists (e.g. Implementation/Validation), attachments.

**Nested links:** traverse per `<project-root>/docs/issue-trackers.md` — real requirements may live on an inner card. Add each ref found to `<ticket-refs>`; every nested ticket contributes to `<expected-outcomes>`.

Extract `<expected-outcomes>` — one entry per acceptance criterion, validation item, or spec requirement the PR must deliver. A ticket that fails to load → record a gap, don't guess its criteria.

Build `<coverage-map>`: each `<expected-outcomes>` entry → the diff files/modules that should deliver it. Flag outcomes with no corresponding change — the PR may not address every linked card.

**Done when:** every linked ticket loaded or gapped · every criterion/validation item in `<expected-outcomes>` · coverage map built with gaps flagged.

### 4. Align Local Branch

Record `<previous-branch>` via `git branch --show-current`. Uncommitted changes → STOP, ask user to stash/commit (interactive question tool when one exists). Check out via git host checkout PR or CLI fallback (`<project-root>/docs/git-hosts.md`); fails → STOP. Restore `<previous-branch>` when testing ends.

### 5. Build the Test Plan

From `<coverage-map>`, write `<test-plan>` with three scenario classes:

- **Delivery** — one user-level scenario per `<expected-outcomes>` entry: entry point, user action, expected observable result (UI state, DB effect, or both).
- **Data-artifact** — one execution scenario per migration, seeder, operation, or script flagged in Step 2: run against a populated DB; assert schema lands and existing rows survive (migrations); expected rows/effects land (seeders, operations, scripts); re-run safe where idempotency is claimed.
- **Regression** — the surface past the diff: direct callers and shared modules traced via code navigation (`<project-root>/docs/code-navigation.md`), each asserting unchanged behavior; project tests covering the changed symbols, run by file/name filter; and a **system pass**: the core journeys every user hits (auth, landing view, primary CRUD flow) exercised even where the diff never touches them.

Order setup-free flows first; note test-data prerequisites per scenario.

**Done when:** every `<expected-outcomes>` entry maps to a delivery scenario · every flagged artifact maps to an execution scenario · regression callers enumerated · system-pass journeys picked.

### 6. Exercise the Change

Run the app per project tooling. Drive each scenario through browser automation (`<project-root>/docs/browser-automation.md`) as a **regular user** — real UI navigation, forms, clicks. No console shortcuts, direct API calls, or DB writes to force state; seed/fixture setup only when a scenario genuinely requires it.

After each scenario, verify **persisted effects** via read-only DB inspection (`<project-root>/docs/database-tools.md`): rows created/updated, status transitions, relationship integrity. Live DB is ground truth — a UI that looks right while the DB is wrong is a failure.

Execute each data-artifact scenario per project tooling against a populated testing database, then inspect live schema and rows. A migration that applies but strands stale columns, or a seeder that runs clean but writes wrong rows, is a failure. Artifact output text is not evidence; the database is.

Regressions: run diff-related tests and lint where they exist, plus every regression scenario and the system pass. A test failure the diff did not cause is still a finding — mark it pre-existing after a cheap check against `<base-branch>`.

Record `<results>` per scenario: `pass`/`fail`/`blocked` with evidence (observed UI state, query results, error text).

**Done when:** every scenario attempted · each `<expected-outcomes>` entry has a result · every artifact executed or `blocked` with reason · related-test results and system pass captured.

### 7. Report Findings

Chat only — never post to the PR. Per `<expected-outcomes>` entry: verified / failed / untested (with reason). Then:

- **Issues** — severity, reproduction steps, evidence, `file:line` where the diff is implicated.
- **Coverage gaps** — linked tickets/cards the PR does not fully address (`<coverage-map>` flags + failed scenarios).
- **Data artifacts** — what each executed, or why `blocked`.
- **Regressions** — adjacent behavior that broke: regression scenarios, test failures (pre-existing vs caused), system-pass failures.
- **Verdict** — `works` only when delivered outcomes pass AND the system still works around them; otherwise `issues found`, one line.

## Notes

- This skill verifies; it never fixes. Issues found → user invokes a fix workflow separately.
- Leave the repo as found: no code edits, no stray commits.
- A scenario that cannot be exercised locally (external service, missing seed) → `blocked` with reason — never `pass` unverified.
- An artifact too destructive to run locally (drops tables, mutates shared infra) → `blocked` with the risk named; review its code and report what verification the deploy needs instead.
