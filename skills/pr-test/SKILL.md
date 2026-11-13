---
name: pr-test
description: "Use when testing or verifying a pull request end-to-end — exercise it as a regular user (UI + database) or through its real entry points (API contracts, workers/jobs, CLI commands), run its migrations/seeders/operations/scripts, verify linked tickets, and prove the system still works past the diff. Reports findings in chat; leaves production, the PR, and the repo untouched; typically post-approval. Args: `quick` for a shallow delivery-only pass, `budget <N>` to cap executed scenarios."
---

## Required `<project-root>/docs` reads

Shell-read before testing the PR; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`; `<project-root>/docs/issue-trackers.md` when the PR references tickets · `<project-root>/docs/database-tools.md` when a scenario provisions or verifies data · `<project-root>/docs/code-navigation.md` on the `shared-surface` flag · `<project-root>/docs/browser-automation.md` at Step 7

Test whether a PR **works**, not whether it reads well — and whether the **system** still works around it, at a depth the diff's risk justifies. A green feature check inside a broken system is a failed test; a system-wide audit behind an isolated change is wasted work.

**Boundary:** the local environment is fully mutable — checkouts, migrations, seeders, operations, browser-driven writes, and local DB writes all execute as part of testing, without user confirmation. Untouched scope: production systems, the PR thread, and the repo's files and history.

**Completion gate:** finish only when every planned scenario has an evidence-backed result — `PASSED`, `FAILED`, `BLOCKED`, or `DEFERRED`. `PASSED` means the executed behavior met every assertion; `FAILED` means it contradicted an assertion; `BLOCKED` means execution never reached the assertions after the Step 6 recovery budget ran out or a needed user action was declined or unavailable — it requires recorded evidence: the missing prerequisite, attempted recoveries and their errors, what would unblock it; `DEFERRED` means the plan cut the scenario by `quick` depth or `<scenario-budget>` — it needs only its recorded reason. Never turn missing evidence into `PASSED` or `FAILED`.

### 1. Interpret Arguments

PR number/URL → `<pr-ref>`; `quick` → `<test-depth>` = `quick` (default `full`); `budget <N>` → `<scenario-budget>` = `<N>` (default uncapped); extra focus areas or constraints → `<additional-context>`; no arguments → derive from conversation or the open PR for the current branch (git host list PRs, `git-hosts.md`).

### 2. Load PR Context

View PR + diff via git host (`git-hosts.md`). Extract `<pr-purpose>` (why — title, body, commit messages), `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`, `<changes>`, `<ticket-refs>` (every issue/card reference in body, commits, or branch name). Unloadable → STOP.

Classify `<changes>` → `<risk-flags>`; each flag escalates only the steps that name it:

- **`data-artifacts`** — migrations, seeders, operations, one-off scripts: code whose only real test is execution against a populated database.
- **`shared-surface`** — config, middleware, providers, base classes, factories, shared modules: blast radius spans the whole app.
- **`client-origin`** — set in Step 3: a linked ticket filed in the customer-facing tracker or carrying a client requester/thread.

No flags → **feature path**: delivery scenarios plus the covering test suite — no caller tracing, no dedicated system pass (reaching the feature as a user already traverses the app shell), no production copies.

### 3. Load Linked Context

For each `<ticket-refs>` entry, load via the issue tracker load tool (`issue-trackers.md`): body and checklists. Load comments, attachment contents, and nested links only on `client-origin` or when the body leaves acceptance criteria ambiguous. If the tracker tool fails, try the configured alternatives, including the authenticated browser. An access boundary requiring the user → ask once for access or the missing ticket content; supplied → resume; declined or unavailable → drop that ticket's outcomes, flag the unloaded ticket in Step 8, and continue with the criteria the loaded sources support.

Extract `<expected-outcomes>` — one entry per acceptance criterion, validation item, or spec requirement the PR must deliver. Resolve missing linked context before finalizing the plan; derive criteria from loaded sources, never guesses.

Classify `<request-source>` per loaded ticket — an unloaded ticket records an unknown source alongside its gap: client-origin (customer-facing tracker or client requester/thread) → set the flag and extract `<client-cases>` — the reported steps, the records/data and module involved, and the symptom or use case in the client's own terms.

Build `<coverage-map>`: each `<expected-outcomes>` entry → the diff files/modules that should deliver it. Flag outcomes with no corresponding change — the PR may not address every linked card.

Flag `<premise-drift>` — an `<expected-outcomes>` or `<client-cases>` entry whose assumed bug path, invariant, or behavior no longer exists in the diff's code (superseding change, misdiagnosed root cause). Drifted entries get no scenario — Step 5 skips them and Step 7 accounting excludes them; report in Step 8 rather than forcing a scenario onto a dead premise.

**Done when:** every linked ticket loaded or dropped with its gap recorded · every criterion/validation item from loaded sources in `<expected-outcomes>` · `<request-source>` classified for loaded tickets, with unknown sources recorded for unloaded tickets · `<client-cases>` extracted where client-origin is established · coverage map built with gaps and premise drift flagged.

### 4. Align Local Branch

Record `<previous-branch>` via `git branch --show-current`. Uncommitted changes → STOP, ask user to stash/commit. Check out via git host checkout PR or CLI fallback (`git-hosts.md`); fails → STOP. Restore `<previous-branch>` when testing ends.

### 5. Build the Test Plan

From `<coverage-map>`, write `<test-plan>` with scenario classes gated by flag:

- **Source** (`client-origin`) — one scenario per `<client-cases>` entry, replaying the client's case as reported: their steps, their data shapes, their flow. A client bug report → the exact reported failure must now pass. An `<expected-outcomes>` entry a source scenario covers needs no duplicate.
- **Delivery** — one user-level scenario per remaining `<expected-outcomes>` entry: entry point, user action, expected observable result (UI state, DB effect, or both).
- **Data-artifact** (`data-artifacts`) — one execution scenario per flagged migration, seeder, operation, or script: run against the populated local dev DB; assert schema lands and existing rows survive (migrations); expected rows/effects land (seeders, operations, scripts); re-run safe where idempotency is claimed.
- **Regression** — feature path: the project tests covering the changed files, run by file/name filter. `shared-surface` adds: direct callers traced via code navigation (`code-navigation.md`), each asserting unchanged behavior, plus a **system pass** — the core journeys every user hits (auth, landing view, primary CRUD flow) exercised even where the diff never touches them.

`quick` depth → build only the feature-path classes — delivery scenarios plus the covering regression suite — regardless of flags; the skipped classes (source, data-artifact, traced-caller, system pass) report as `DEFERRED` in Step 8. When a `quick` scenario fails where a skipped class applied, escalate that area to `full`.

Order execution source → delivery → data-artifact → regression, setup-free flows first; a scenario asserting another scenario's effects runs after it. `<scenario-budget>` cuts the tail of that order: mark cut scenarios `DEFERRED` with the budget as the reason.

Assign each scenario a stable ID and record its source criterion/artifact/caller, prerequisites, actual execution entry point, and explicit assertions. Include applicable negative, permission, boundary, and legacy-data cases required by the acceptance criteria or changed behavior. A missing implementation still needs an execution scenario that demonstrates the missing behavior.

**Done when:** every non-`<premise-drift>` `<client-cases>` entry maps to a source scenario (a delivery scenario under `quick` depth) · every non-`<premise-drift>` `<expected-outcomes>` entry maps to a scenario · every flagged artifact maps to an execution scenario (`DEFERRED` under `quick` depth or past `<scenario-budget>`) · regression scope set per path · every scenario has an ID, execution entry point, prerequisites, and assertions. This is complete coverage of the scoped scenarios, not a claim of exhaustive application or code coverage.

### 6. Prepare Every Scenario

Treat missing local data, tooling, runtime dependencies, credentials, workers, and services as setup work. Inspect the configured project tools and environment, start the required local processes, and diagnose startup failures. For each scenario, identify its required records, tenant context, relationships, permissions, and reproduction-critical values; inspect the live local DB to find what is missing.

Build the smallest faithful fixture: use existing seeds/factories where suitable; otherwise create the missing records or adjust dedicated test records through application tooling or direct local DB inserts/updates. Preserve the reported failure's relevant data shapes and relationships, including edge-case or legacy states that ordinary factories omit. Production copies are an option only for source scenarios or when local construction cannot reproduce the required state; unavailable production access alone does not block a locally constructible scenario.

Local provisioning is part of testing, not a repository fix: create fixtures in the local DB without editing tracked files. Additive local writes need no further permission; ask before overwriting or deleting existing local data. Keep setup separate from the behavior under test: prepare prerequisites, then exercise the real user flow or artifact in Step 7 rather than writing its expected result into the DB.

Production copy rules (`database-tools.md`):

- Confirm the source is production and the destination is local before importing.
- Copy only the records needed for the scenario, including required relationships and tenant context. Preserve reproduction-critical values; mask unrelated sensitive data and exclude secrets from logs, reports, and version control.
- Isolate local mail, queues, webhooks, and external integrations before exercising copied data so reproduction cannot affect real users or services.
- Write only to local — never to production. Ask before overwriting or deleting existing local data.

**Recovery loop:** when setup or execution fails, capture the exact error, inspect its cause using the available tools/codebase/logs, repair local setup within the boundary, and retry the same scenario. Try applicable fixture methods above and configured tool alternatives rather than treating the first failure as a stopping point. Cap recovery at three attempts per scenario — an attempt is a diagnosed cause, a repair, and a retry; exhausted → record the scenario `BLOCKED` and move on. Continue independent scenarios while resolving the prerequisite; return to every unfinished ID before reporting.

For external dependencies, use an available sandbox or a controlled local substitute at the external boundary while executing the real application path. Record what was substituted and what the evidence proves. A substitute can prove application handling of a response; it cannot prove the real provider accepted a request. A criterion requiring the real provider still requires that execution.

If further progress needs user access, credentials, approval, or a human-only action, ask once for that specific action with the exact error and attempted recoveries. Supplied → resume the scenario; declined or unanswered → record it `BLOCKED` and move on. Safety boundaries remain binding: use an isolated disposable local copy for destructive artifacts only with any required approval, and never act on production or shared infrastructure.

**Done when:** every non-`DEFERRED` scenario's runtime prerequisites are ready · required records exist locally · inspection confirms reproduction-critical values and relationships. Unmet prerequisites keep their scenarios in the recovery loop until ready or `BLOCKED`.

### 7. Exercise the Change

Independent scenarios run in parallel — dispatch each scenario's setup and execution to a subagent where the platform supports it, but only at ≥3 independent scenarios (the `subagents` parallel-time-win gate); fewer run inline. Scenarios sharing mutable local state (the same records, tenant, browser session, or fixture) stay serial.

Run the app per project tooling. Drive UI scenarios through browser automation (`browser-automation.md`) as a **regular user** — real UI navigation, forms, clicks. Exercise non-UI scenarios through their actual entry points: CLI command, worker/job dispatch, API for an API contract, or the artifact runner. For a UI scenario, console shortcuts, direct API calls, and DB writes cannot replace the user action — Step 6 provisions prerequisites, never the expected result.

Verify **persisted effects** via read-only DB inspection (`database-tools.md`) only where an outcome asserts a data effect — rows created/updated, status transitions, relationship integrity. A scenario asserting only UI state needs no query. Live DB is ground truth — a UI that looks right while the DB is wrong is a failure.

Execute each data-artifact scenario per project tooling against the populated local dev database — the same DB the browser scenarios exercise, so artifact effects are visible to UI checks — then inspect live schema and rows. A migration that applies but strands stale columns, or a seeder that runs clean but writes wrong rows, is a failure. Artifact output text is not evidence; the database is.

Regressions: run diff-related tests and lint where they exist; `shared-surface` adds the traced-caller scenarios and the system pass. A test failure the diff did not cause is still a finding — mark it pre-existing after a cheap check against `<base-branch>`.

Capture a confirming artifact per scenario into `<evidence-dir>` — a temp dir outside the repo (e.g. `$(mktemp -d)/pr-test-<pr-number>`) so the repo stays untouched — named `<scenario-id>-<slug>`: a screenshot of each asserted UI state, saved command/runner output, the read-only query and its result, or the blocking error capture. Record artifact paths in `<results>` — Step 8 cites them as evidence. An artifact confirms an executed flow; it never substitutes for executing it.

Record `<results>` by scenario ID: execution steps/command, expected assertions, observed results, evidence (the `<evidence-dir>` path, read-only query and result, test output, or error text), and `PASSED`/`FAILED`/`BLOCKED`. Code inspection and a green test suite support diagnosis; neither substitutes for an unexecuted user flow or artifact. A runtime error reached through the real scenario is failure evidence; a tool/setup error preventing that execution returns to Step 6.

Before reporting, reconcile `<coverage-map>`, `<test-plan>`, and `<results>`: every non-`<premise-drift>` criterion, client case, artifact, and required regression maps to scenario IDs; each ID carries a final result — `PASSED`/`FAILED` with evidence, `BLOCKED` with its recovery record, `DEFERRED` with its reason; every referenced ticket loaded or its unloaded state recorded; planned count = `PASSED` + `FAILED` + `BLOCKED` + `DEFERRED`. Reopen missing rows, partial assertions, and evidence gaps, then execute them. Keep scenarios in the ledger when difficult; completion requires their results, not their removal.

**Done when:** every planned scenario holds a result — executed with every assertion evaluated and its confirming artifact captured, or `BLOCKED`/`DEFERRED` recorded per their rules · each non-`<premise-drift>` `<expected-outcomes>` entry maps to scenario results · every artifact and required regression executed or recorded `BLOCKED`/`DEFERRED` · reconciliation passes.

### 8. Report Findings

Chat only — never post to the PR.

- **Verdict** — `PASSED` when every executed scenario passed, no `BLOCKED` scenarios remain, and every referenced ticket loaded · `FAILED` when at least one executed scenario failed · `INCOMPLETE` when nothing failed but `BLOCKED` scenarios remain or a referenced ticket stayed unloaded. Include total, passed, failed, and blocked counts; `DEFERRED` items list separately.
- **Scenario results** — one row per scenario ID: criterion/artifact/regression, result, expected vs. observed behavior, and concrete evidence — the `<evidence-dir>` path for executed scenarios. `BLOCKED` rows carry the missing prerequisite and attempted recoveries; `DEFERRED` rows carry the `quick` depth or `<scenario-budget>` that cut them. Include passing scenarios, artifact effects, and regression checks rather than hiding them in a count.
- **Failures** — reason, evidence, and `file:line` where the diff is implicated; distinguish caused from pre-existing failures. Include linked requirements the PR does not deliver, backed by their executed scenarios; name any referenced ticket that stayed unloaded — its criteria unknown, not absent. Report `<premise-drift>` here — the assumed bug path or behavior gone, what superseded it — distinct from scenario `FAILED`.
- **Test boundaries** — identify sandbox/substitute use and the exact behavior it proved; name the scenario classes a `quick` run or `<scenario-budget>` never exercised. Claim only what the execution demonstrates.

## Notes

- Verifies, never fixes — issues found → user invokes a fix workflow; leave the repo as found (no edits, no stray commits).
- Final findings require the completion gate. `BLOCKED` and `DEFERRED` are first-class results — the report names each with its reason and unblock path, never a silent skip. Honest incompletion never becomes a fabricated `PASSED` or `FAILED`.
