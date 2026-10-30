---
name: pr-test
description: "Use when testing or verifying a pull request end-to-end — exercise the change as a regular user (UI + database), execute the migrations, seeders, operations, and scripts it ships, verify every linked ticket/card is delivered, and prove the whole system still works past the diff. READ-ONLY; reports findings in chat. Often run after the PR is approved."
---

## Required `<project-root>/docs` reads

Read these project-root spec files before testing the PR (use shell `cat`/`ls` — they may be in `.gitignore`, invisible to built-in search). Missing file → fall back to native tools, note the gap; never invent contents.

- `<project-root>/docs/browser-automation.md`
- `<project-root>/docs/code-navigation.md`
- `<project-root>/docs/database-tools.md`
- `<project-root>/docs/git-hosts.md`
- `<project-root>/docs/issue-trackers.md`

Test whether a PR **works**, not whether it reads well — and whether the **system** still works around it. The diff marks where the change landed; the blast radius is every behavior, caller, and data artifact it can break. A green feature check inside a broken system is a failed test. Load the PR's purpose and linked tickets/specs, check out the branch, drive the system as a regular user, execute the data artifacts it ships, verify persisted effects in the database, confirm coverage of every related card, and sweep the system for regressions. READ-ONLY — no code changes, no commits, nothing posted to the PR. Findings go to chat.

### 1. Interpret Arguments

- PR number/URL → `<pr-ref>`; extra focus areas or constraints → `<additional-context>`; no arguments → derive from conversation or the open PR for the current branch (git host list PRs, see `<project-root>/docs/git-hosts.md` in the project root).

### 2. Load PR Context

View the PR via git host view PR + PR diff (see `<project-root>/docs/git-hosts.md`). Extract `<pr-purpose>` (why it was created — title, body, commit messages), `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`, `<changes>`, and `<ticket-refs>` (every issue/card reference in the body, commits, or branch name — on the git host or any external tracker). Unloadable PR → STOP.

Classify `<changes>`. Flag **data artifacts** — migrations, seeders, operations, one-off scripts — code whose only real test is execution against a populated database. Flag shared and infra files (config, middleware, providers, base classes, factories) whose blast radius spans the whole app. Both feed the test plan; neither is covered by a UI scenario alone.

### 3. Load Linked Context

For each `<ticket-refs>` entry, load the ticket via the issue tracker tool (see `<project-root>/docs/issue-trackers.md`): body, every comment, checklists (e.g. Implementation/Validation sections), attachments.

**Follow nested links.** A PR commonly links to a git host issue that itself links to a card on another platform — the real requirements live on the inner card. After loading each ticket, scan its body, comments, checklists, and attachments for references to other issues/cards on any platform. Load each new reference, repeat the scan on it, and add it to `<ticket-refs>` until no new refs surface. Dedupe visited refs so cross-linked tickets don't loop. Every nested ticket contributes to `<expected-outcomes>`.

Extract `<expected-outcomes>` — one entry per acceptance criterion, validation item, or spec requirement the PR must deliver. A linked ticket that fails to load → record a gap, don't guess its criteria.

Build `<coverage-map>`: map each `<expected-outcomes>` entry to the diff files/modules that should deliver it. Flag outcomes with no corresponding change — the PR may not address every linked card.

**Done when:** every linked ticket loaded or gapped · every acceptance criterion/validation item in `<expected-outcomes>` · coverage map built with gaps flagged.

### 4. Align Local Branch

Record `<previous-branch>` via `git branch --show-current`. Uncommitted changes → STOP, ask user to stash/commit via the interactive question tool when one exists. Check out via git host checkout PR or CLI fallback per `<project-root>/docs/git-hosts.md`. Checkout fails → STOP. Restore `<previous-branch>` when testing ends.

### 5. Build the Test Plan

From `<coverage-map>`, write `<test-plan>` with three scenario classes:

- **Delivery** — one user-level scenario per `<expected-outcomes>` entry: entry point, user action, expected observable result (UI state, DB effect, or both).
- **Data-artifact** — one execution scenario per migration, seeder, operation, or script flagged in Step 2: run it against a populated database, assert the schema lands and existing rows survive (migrations), assert the expected rows or effects land (seeders, operations, scripts), and assert re-running is safe where the artifact claims idempotency.
- **Regression** — the surface past the diff: direct callers and shared modules traced via code navigation (see `<project-root>/docs/code-navigation.md`), each a scenario asserting unchanged behavior; the project's own test suite run in full — the whole suite, not the files adjacent to the diff; and a **system pass**: the core journeys every user hits (auth, the landing view, the app's primary CRUD flow) exercised even where the diff never touches them.

Order scenarios so setup-free flows run first; note test-data prerequisites per scenario.

**Done when:** every `<expected-outcomes>` entry maps to a delivery scenario · every flagged data artifact maps to an execution scenario · regression callers enumerated and named · system pass journeys picked.

### 6. Exercise the Change

Run the app per the project's tooling. Drive each `<test-plan>` scenario through browser automation (see `<project-root>/docs/browser-automation.md`) as a **regular user** — real UI navigation, forms, clicks. No console shortcuts, no direct API calls, no DB writes to force state; seed/fixture setup only when a scenario genuinely requires it.

After each scenario, verify **persisted effects** via read-only database inspection (see `<project-root>/docs/database-tools.md`): rows created/updated, status transitions, relationship integrity. The live database is ground truth — a UI that looks right while the DB is wrong is a failure.

Execute each data-artifact scenario per the project's tooling — run the migration, invoke the operation, run the seeder or script — against a populated testing database, then inspect the live schema and rows. A migration that applies but strands stale columns or nulls, or a seeder that runs clean but writes wrong rows, is a failure. Artifact output text is not evidence; the database is.

Regressions: run the project's test/lint suite in full where it exists — a filtered subset proves nothing about the rest — plus every regression scenario and the system pass from Step 5. A suite failure the diff did not cause is still a finding: mark it pre-existing after a cheap check against `<base-branch>`.

Record `<results>` per scenario: `pass` / `fail` / `blocked`, with evidence (observed UI state, query results, error text).

**Done when:** every scenario attempted · each `<expected-outcomes>` entry has a result · every data artifact executed or `blocked` with reason · full-suite results and system pass captured.

### 7. Report Findings

Chat only — never post to the PR. Per `<expected-outcomes>` entry: verified / failed / untested (with reason). Then:

- **Issues** — severity, reproduction steps, evidence, `file:line` where the diff is implicated.
- **Coverage gaps** — linked tickets/cards the PR does not fully address (from `<coverage-map>` flags and failed scenarios).
- **Data artifacts** — what each migration, seeder, operation, and script did when executed, or why it was `blocked`.
- **Regressions** — adjacent behavior that broke: regression scenarios, suite failures (marked pre-existing vs caused), system-pass failures.
- **Verdict** — `works` only when the delivered outcomes pass AND the system still works around them; otherwise `issues found`, one line.

## Notes

- This skill verifies; it never fixes. Issues found → user invokes a fix workflow separately.
- Leave the repo as found: no code edits, no stray commits.
- A scenario that cannot be exercised locally (external service, missing seed) → mark `blocked` with the reason, never mark `pass` unverified.
- A data artifact too destructive to run locally (drops tables, mutates shared infra) → mark `blocked` with the risk named; review its code and report what verification the deploy will need instead.
