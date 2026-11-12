---
name: pr-test
description: "Use when testing or verifying a pull request end-to-end — exercise it as a regular user (UI + database) or through its real entry points (API contracts, workers/jobs, CLI commands), run its migrations/seeders/operations/scripts, verify linked tickets, and prove the system still works past the diff. Reports findings in chat; leaves production, the PR, and the repo untouched; typically post-approval."
---

## Required `<project-root>/docs` reads

Shell-read before testing the PR; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`; `<project-root>/docs/issue-trackers.md` when the PR references tickets · `<project-root>/docs/database-tools.md` when a scenario provisions or verifies data · `<project-root>/docs/code-navigation.md` on the `shared-surface` flag · `<project-root>/docs/browser-automation.md` at Step 7

Test whether a PR **works**, not whether it reads well — and whether the **system** still works around it, at a depth the diff's risk justifies. A green feature check inside a broken system is a failed test; a system-wide audit behind an isolated change is wasted work.

**Boundary:** the local environment is fully mutable — checkouts, migrations, seeders, operations, browser-driven writes, and local DB writes all execute as part of testing, without user confirmation. Untouched scope: production systems, the PR thread, and the repo's files and history.

**Completion gate:** finish only when every planned scenario has an evidence-backed `PASSED` or `FAILED` result. `PASSED` means the executed behavior met every assertion; `FAILED` means the executed behavior contradicted an assertion. Missing setup, access, tools, or execution is unfinished work, not a product result. Resolve it through the recovery loop in Step 6; request the specific user action when a genuine permission/access boundary requires it. Keep the run incomplete until execution resumes. Never turn missing evidence into a verdict.

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

For each `<ticket-refs>` entry, load via the issue tracker load tool (`issue-trackers.md`): body and checklists. Load comments, attachment contents, and nested links only on `client-origin` or when the body leaves acceptance criteria ambiguous. If the tracker tool fails, try the configured alternatives, including the authenticated browser. An access boundary requiring the user → request access or the missing ticket content; resume when available.

Extract `<expected-outcomes>` — one entry per acceptance criterion, validation item, or spec requirement the PR must deliver. Resolve missing linked context before finalizing the plan; derive criteria from loaded sources, never guesses.

Classify `<request-source>` per ticket: client-origin (customer-facing tracker or client requester/thread) → set the flag and extract `<client-cases>` — the reported steps, the records/data and module involved, and the symptom or use case in the client's own terms.

Build `<coverage-map>`: each `<expected-outcomes>` entry → the diff files/modules that should deliver it. Flag outcomes with no corresponding change — the PR may not address every linked card.

**Done when:** every linked ticket loaded · every criterion/validation item in `<expected-outcomes>` · `<request-source>` classified per ticket, `<client-cases>` extracted where client-origin · coverage map built with gaps flagged.

### 4. Align Local Branch

Record `<previous-branch>` via `git branch --show-current`. Uncommitted changes → STOP, ask user to stash/commit. Check out via git host checkout PR or CLI fallback (`git-hosts.md`); fails → STOP. Restore `<previous-branch>` when testing ends.

### 5. Build the Test Plan

From `<coverage-map>`, write `<test-plan>` with scenario classes gated by flag:

- **Source** (`client-origin`) — one scenario per `<client-cases>` entry, replaying the client's case as reported: their steps, their data shapes, their flow. A client bug report → the exact reported failure must now pass. An `<expected-outcomes>` entry a source scenario covers needs no duplicate.
- **Delivery** — one user-level scenario per remaining `<expected-outcomes>` entry: entry point, user action, expected observable result (UI state, DB effect, or both).
- **Data-artifact** (`data-artifacts`) — one execution scenario per flagged migration, seeder, operation, or script: run against the populated local dev DB; assert schema lands and existing rows survive (migrations); expected rows/effects land (seeders, operations, scripts); re-run safe where idempotency is claimed.
- **Regression** — feature path: the project tests covering the changed files, run by file/name filter. `shared-surface` adds: direct callers traced via code navigation (`code-navigation.md`), each asserting unchanged behavior, plus a **system pass** — the core journeys every user hits (auth, landing view, primary CRUD flow) exercised even where the diff never touches them.

Assign each scenario a stable ID and record its source criterion/artifact/caller, prerequisites, actual execution entry point, and explicit assertions. Include applicable negative, permission, boundary, and legacy-data cases required by the acceptance criteria or changed behavior. Order setup-free flows first. A missing implementation still needs an execution scenario that demonstrates the missing behavior.

**Done when:** every `<client-cases>` entry maps to a source scenario · every `<expected-outcomes>` entry maps to a scenario · every flagged artifact maps to an execution scenario · regression scope set per path · every scenario has an ID, execution entry point, prerequisites, and assertions. This is complete coverage of the scoped scenarios, not a claim of exhaustive application or code coverage.

### 6. Prepare Every Scenario

Treat missing local data, tooling, runtime dependencies, credentials, workers, and services as setup work. Inspect the configured project tools and environment, start the required local processes, and diagnose startup failures. For each scenario, identify its required records, tenant context, relationships, permissions, and reproduction-critical values; inspect the live local DB to find what is missing.

Build the smallest faithful fixture: use existing seeds/factories where suitable; otherwise create the missing records or adjust dedicated test records through application tooling or direct local DB inserts/updates. Preserve the reported failure's relevant data shapes and relationships, including edge-case or legacy states that ordinary factories omit. Production copies are an option only for source scenarios or when local construction cannot reproduce the required state; unavailable production access alone does not block a locally constructible scenario.

Local provisioning is part of testing, not a repository fix: create fixtures in the local DB without editing tracked files. Additive local writes need no further permission; ask before overwriting or deleting existing local data. Keep setup separate from the behavior under test: prepare prerequisites, then exercise the real user flow or artifact in Step 7 rather than writing its expected result into the DB.

Production copy rules (`database-tools.md`):

- Confirm the source is production and the destination is local before importing.
- Copy only the records needed for the scenario, including required relationships and tenant context. Preserve reproduction-critical values; mask unrelated sensitive data and exclude secrets from logs, reports, and version control.
- Isolate local mail, queues, webhooks, and external integrations before exercising copied data so reproduction cannot affect real users or services.
- Write only to local — never to production. Ask before overwriting or deleting existing local data.

**Recovery loop:** when setup or execution fails, capture the exact error, inspect its cause using the available tools/codebase/logs, repair local setup within the boundary, and retry the same scenario. Try applicable fixture methods above and configured tool alternatives rather than treating the first failure as a stopping point. Continue independent scenarios while resolving the prerequisite; return to every unfinished ID before reporting.

For external dependencies, use an available sandbox or a controlled local substitute at the external boundary while executing the real application path. Record what was substituted and what the evidence proves. A substitute can prove application handling of a response; it cannot prove the real provider accepted a request. A criterion requiring the real provider still requires that execution.

If further progress needs user access, credentials, approval, or a human-only action, ask for that specific action with the exact error and attempted recoveries. Keep the affected scenario pending and the run incomplete; resume it after the prerequisite is supplied. Safety boundaries remain binding: use an isolated disposable local copy for destructive artifacts only with any required approval, and never act on production or shared infrastructure.

**Done when:** every scenario's runtime prerequisites are ready · required records exist locally · inspection confirms reproduction-critical values and relationships. Unmet prerequisites keep this step open.

### 7. Exercise the Change

Run the app per project tooling. Drive UI scenarios through browser automation (`browser-automation.md`) as a **regular user** — real UI navigation, forms, clicks. Exercise non-UI scenarios through their actual entry points: CLI command, worker/job dispatch, API for an API contract, or the artifact runner. For a UI scenario, console shortcuts, direct API calls, and DB writes cannot replace the user action — Step 6 provisions prerequisites, never the expected result.

Verify **persisted effects** via read-only DB inspection (`database-tools.md`) only where an outcome asserts a data effect — rows created/updated, status transitions, relationship integrity. A scenario asserting only UI state needs no query. Live DB is ground truth — a UI that looks right while the DB is wrong is a failure.

Execute each data-artifact scenario per project tooling against the populated local dev database — the same DB the browser scenarios exercise, so artifact effects are visible to UI checks — then inspect live schema and rows. A migration that applies but strands stale columns, or a seeder that runs clean but writes wrong rows, is a failure. Artifact output text is not evidence; the database is.

Regressions: run diff-related tests and lint where they exist; `shared-surface` adds the traced-caller scenarios and the system pass. A test failure the diff did not cause is still a finding — mark it pre-existing after a cheap check against `<base-branch>`.

Record `<results>` by scenario ID: execution steps/command, expected assertions, observed results, evidence (UI state/capture, read-only query and result, test output, or error text), and `PASSED`/`FAILED`. Code inspection and a green test suite support diagnosis; neither substitutes for an unexecuted user flow or artifact. A runtime error reached through the real scenario is failure evidence; a tool/setup error preventing that execution returns to Step 6.

Before reporting, reconcile `<coverage-map>`, `<test-plan>`, and `<results>`: every criterion, client case, artifact, and required regression maps to executed scenario IDs; each ID has evidence and a result; planned count = `PASSED` count + `FAILED` count. Reopen missing rows, partial assertions, and evidence gaps, then execute them. Keep scenarios in the ledger when difficult; completion requires their results, not their removal.

**Done when:** every planned scenario executed with every assertion evaluated · each `<expected-outcomes>` entry has evidence-backed results · every artifact executed and effects inspected · required regressions executed · reconciliation passes.

### 8. Report Findings

Chat only — never post to the PR.

- **Verdict** — `PASSED` only when every planned scenario passed; `FAILED` when at least one executed scenario failed and all planned scenarios have results. Include total, passed, and failed counts.
- **Scenario results** — one row per scenario ID: criterion/artifact/regression, `PASSED` or `FAILED`, expected vs. observed behavior, and concrete evidence. Include passing scenarios, artifact effects, and regression checks rather than hiding them in a count.
- **Failures** — reason, evidence, and `file:line` where the diff is implicated; distinguish caused from pre-existing failures. Include linked requirements the PR does not deliver, backed by their executed scenarios.
- **Test boundaries** — identify sandbox/substitute use and the exact behavior it proved. Claim only what the execution demonstrates.

## Notes

- Verifies, never fixes — issues found → user invokes a fix workflow; leave the repo as found (no edits, no stray commits).
- Final findings require the completion gate. An unresolved prerequisite calls for a specific unblock request, not a finished report with skipped, unverified, not-checked, or blocked scenarios. Honest incompletion never becomes a fabricated `PASSED` or `FAILED`.
