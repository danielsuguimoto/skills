---
name: pr-test
description: "Use when testing or verifying a pull request end-to-end — exercise it as a regular user (UI + database), run its migrations/seeders/operations/scripts, verify linked tickets, and prove the system still works past the diff. Reports findings in chat; leaves production, the PR, and the repo untouched; typically post-approval."
---

## Required `<project-root>/docs` reads

Shell-read before testing the PR; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`; `<project-root>/docs/issue-trackers.md` when the PR references tickets · `<project-root>/docs/database-tools.md` when a scenario provisions or verifies data · `<project-root>/docs/code-navigation.md` on the `shared-surface` flag · `<project-root>/docs/browser-automation.md` at Step 7

Test whether a PR **works**, not whether it reads well — and whether the **system** still works around it, at a depth the diff's risk justifies. A green feature check inside a broken system is a failed test; a system-wide audit behind an isolated change is wasted work.

**Boundary:** the local environment is fully mutable — checkouts, migrations, seeders, operations, browser-driven writes, and local DB writes all execute as part of testing, without user confirmation. Untouched scope: production systems, the PR thread, and the repo's files and history.

### 1. Interpret Arguments

PR number/URL → `<pr-ref>`; extra focus areas or constraints → `<additional-context>`; no arguments → derive from conversation or the open PR for the current branch (git host list PRs, `git-hosts.md`).

### 2. Load PR Context

View PR + diff via git host (`git-hosts.md`). Extract `<pr-purpose>` (why — title, body, commit messages), `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`, `<changes>`, `<ticket-refs>` (every issue/card reference in body, commits, or branch name). Unloadable → STOP.

Classify `<changes>` → `<risk-flags>`; each flag escalates only the steps that name it:

- **`data-artifacts`** — migrations, seeders, operations, one-off scripts: code whose only real test is execution against a populated database.
- **`shared-surface`** — config, middleware, providers, base classes, factories, shared modules: blast radius spans the whole app.
- **`client-origin`** — set in Step 3: a linked ticket filed in the customer-facing tracker or carrying a client requester/thread.

No flags → **feature path**: delivery scenarios plus the covering test suite — no caller tracing, no dedicated system pass (reaching the feature as a user already traverses the app shell), no production copies.

### 3. Load Linked Context

For each `<ticket-refs>` entry, load via the issue tracker load tool (`issue-trackers.md`): body and checklists. Load comments, attachment contents, and nested links only on `client-origin` or when the body leaves acceptance criteria ambiguous. A ref the tracker tooling cannot load → gap, no browser fallback.

Extract `<expected-outcomes>` — one entry per acceptance criterion, validation item, or spec requirement the PR must deliver. A ticket that fails to load → record a gap, don't guess its criteria.

Classify `<request-source>` per ticket: client-origin (customer-facing tracker or client requester/thread) → set the flag and extract `<client-cases>` — the reported steps, the records/data and module involved, and the symptom or use case in the client's own terms.

Build `<coverage-map>`: each `<expected-outcomes>` entry → the diff files/modules that should deliver it. Flag outcomes with no corresponding change — the PR may not address every linked card.

**Done when:** every linked ticket loaded or gapped · every criterion/validation item in `<expected-outcomes>` · `<request-source>` classified per ticket, `<client-cases>` extracted where client-origin · coverage map built with gaps flagged.

### 4. Align Local Branch

Record `<previous-branch>` via `git branch --show-current`. Uncommitted changes → STOP, ask user to stash/commit. Check out via git host checkout PR or CLI fallback (`git-hosts.md`); fails → STOP. Restore `<previous-branch>` when testing ends.

### 5. Build the Test Plan

From `<coverage-map>`, write `<test-plan>` with scenario classes gated by flag:

- **Source** (`client-origin`) — one scenario per `<client-cases>` entry, replaying the client's case as reported: their steps, their data shapes, their flow. A client bug report → the exact reported failure must now pass. An `<expected-outcomes>` entry a source scenario covers needs no duplicate.
- **Delivery** — one user-level scenario per remaining `<expected-outcomes>` entry: entry point, user action, expected observable result (UI state, DB effect, or both).
- **Data-artifact** (`data-artifacts`) — one execution scenario per flagged migration, seeder, operation, or script: run against the populated local dev DB; assert schema lands and existing rows survive (migrations); expected rows/effects land (seeders, operations, scripts); re-run safe where idempotency is claimed.
- **Regression** — feature path: the project tests covering the changed files, run by file/name filter. `shared-surface` adds: direct callers traced via code navigation (`code-navigation.md`), each asserting unchanged behavior, plus a **system pass** — the core journeys every user hits (auth, landing view, primary CRUD flow) exercised even where the diff never touches them.

Order setup-free flows first; note test-data prerequisites per scenario.

**Done when:** every `<client-cases>` entry maps to a source scenario · every `<expected-outcomes>` entry maps to a scenario · every flagged artifact maps to an execution scenario · regression scope set per path.

### 6. Provision Test Data

Before exercising, verify the local DB holds the records each scenario needs — the client case's data shapes, tenant context, required relationships. Missing → provision: seeds/factories for generic data; production copies only for source scenarios or cases seeds/factories cannot reproduce.

Production copy rules (`database-tools.md`):

- Confirm the source is production and the destination is local before importing.
- Copy only the records needed for the scenario, including required relationships and tenant context. Preserve reproduction-critical values; mask unrelated sensitive data and exclude secrets from logs, reports, and version control.
- Isolate local mail, queues, webhooks, and external integrations before exercising copied data so reproduction cannot affect real users or services.
- Write only to local — never to production. Ask before overwriting or deleting existing local data.

**Done when:** every scenario's required records exist locally — seeded, copied, or already present.

### 7. Exercise the Change

Run the app per project tooling. Drive each scenario through browser automation (`browser-automation.md`) as a **regular user** — real UI navigation, forms, clicks. No console shortcuts, direct API calls, or DB writes to force state — Step 6 provisions whatever data a scenario requires.

Verify **persisted effects** via read-only DB inspection (`database-tools.md`) only where an outcome asserts a data effect — rows created/updated, status transitions, relationship integrity. A scenario asserting only UI state needs no query. Live DB is ground truth — a UI that looks right while the DB is wrong is a failure.

Execute each data-artifact scenario per project tooling against the populated local dev database — the same DB the browser scenarios exercise, so artifact effects are visible to UI checks — then inspect live schema and rows. A migration that applies but strands stale columns, or a seeder that runs clean but writes wrong rows, is a failure. Artifact output text is not evidence; the database is.

Regressions: run diff-related tests and lint where they exist; `shared-surface` adds the traced-caller scenarios and the system pass. A test failure the diff did not cause is still a finding — mark it pre-existing after a cheap check against `<base-branch>`.

Record `<results>` per scenario: `pass`/`fail`/`blocked` with evidence (observed UI state, query results, error text).

**Done when:** every scenario attempted · each `<expected-outcomes>` entry has a result · every artifact executed or `blocked` with reason · regression results captured.

### 8. Report Findings

Chat only — never post to the PR.

- **Verdict** — `works` when delivered outcomes pass and exercised regressions hold; otherwise `issues found`, one line.
- **Outcome failures** — one line per failed/untested `<expected-outcomes>` entry: reason, evidence, `file:line` where the diff is implicated. Passing outcomes report as a count.
- **Coverage gaps** — linked tickets/cards the PR does not fully address.
- **Artifacts & regressions** — only when non-empty: what each artifact executed or why `blocked`; adjacent breakage, marked pre-existing or caused.

## Notes

- Verifies, never fixes — issues found → user invokes a fix workflow; leave the repo as found (no edits, no stray commits).
- A scenario that cannot be exercised locally (external-service dependency, data not provisionable from seeds, factories, or production copies) → `blocked` with reason — never `pass` unverified.
- An artifact too destructive to run locally (drops tables, mutates shared infra) → `blocked` with the risk named; review its code and report what verification the deploy needs instead.
