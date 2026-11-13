---
name: manual-test
description: "Use when the user tests a pull request themselves with agent guidance — prepare the local database to reproduce its scenarios, then walk the user step-by-step through exercising the change and verifying linked tickets while you check persisted effects. Fits hands-on verification, human-only flows (auth/2FA, devices, live sessions), or missing browser automation; the user runs everything the PR ships — UI flows, migrations, operations, scripts — while the agent sets up, guides, and verifies."
---

## Required `<project-root>/docs` reads

Shell-read before preparing the test; may be gitignored — missing → note gap, never invent: `<project-root>/docs/code-navigation.md`, `<project-root>/docs/database-tools.md`, `<project-root>/docs/git-hosts.md`; `<project-root>/docs/issue-trackers.md` (only when the PR references tickets/cards)

Test whether a PR **works** with the user as the driver. The agent builds the context, provisions the data, and hands the user exact steps to perform — UI actions and the commands for any data artifacts; the user reports what they observe; the agent verifies persisted effects in the database. Same bar as `pr-test`, different driver — a UI that looks right while the DB is wrong is a failure.

**Boundary:** the local environment is fully mutable for provisioning — checkouts, generic seeders/factories, production copies, and local DB writes all execute as part of setup, without user confirmation. Everything the PR ships is a test action and belongs to the user: UI flows, migrations, operations, seeders, and scripts alike. The agent prepares the pre-artifact state and verifies the after-state; it never drives the UI and never runs a flagged data artifact (that is `pr-test`). Untouched scope: production systems, the PR thread, and the repo's files and history.

### 1. Interpret Arguments

PR number/URL → `<pr-ref>`; extra focus areas or constraints → `<additional-context>`; no arguments → derive from conversation or the open PR for the current branch (git host list PRs, `git-hosts.md`).

### 2. Load PR Context

View PR + diff via git host (`git-hosts.md`). Extract `<pr-purpose>` (why — title, body, commit messages), `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`, `<changes>`, `<ticket-refs>` (every issue/card reference in body, commits, or branch name — git host or external tracker). Unloadable → STOP.

Classify `<changes>`. Flag **data artifacts** — migrations, seeders, operations, one-off scripts — code whose only real test is execution against a populated database. Flag shared/infra files (config, middleware, providers, base classes, factories) — blast radius spans the whole app. Both feed the test plan; neither is covered by a UI scenario alone.

### 3. Load Linked Context

For each `<ticket-refs>` entry, load via the issue tracker load tool (`issue-trackers.md`): body, every comment, checklists, attachments; traverse nested links per the spec — every nested ticket feeds `<expected-outcomes>`. Every ticket, spec, and card load — top-level and nested — goes through the tracker tooling documented there alone. A ref the tracker tooling cannot load → gap per below, no browser fallback.

Extract `<expected-outcomes>` — one entry per acceptance criterion, validation item, or spec requirement the PR must deliver. A ticket that fails to load → record a gap, don't guess its criteria.

Classify `<request-source>` per ticket: **client-origin** — filed in the customer-facing tracker or carrying a client requester/thread (client bug reports, client-specific feature requests) — or **internal**. Client-origin → extract `<client-cases>`: the reported steps, the records/data and module involved, and the symptom or use case in the client's own terms.

Build `<coverage-map>`: each `<expected-outcomes>` entry → the diff files/modules that should deliver it. Flag outcomes with no corresponding change — the PR may not address every linked card.

Flag `<premise-drift>` — an `<expected-outcomes>` or `<client-cases>` entry whose assumed bug path, invariant, or behavior no longer exists in the diff's code (superseding change, misdiagnosed root cause); report in Step 8 rather than forcing a scenario onto a dead premise.

**Done when:** every linked ticket loaded or gapped · every criterion/validation item in `<expected-outcomes>` · `<request-source>` classified per ticket, `<client-cases>` extracted where client-origin · coverage map built with gaps and premise drift flagged.

### 4. Align Local Branch

Record `<previous-branch>` via `git branch --show-current`. Uncommitted changes → STOP, ask user to stash/commit. Check out via git host checkout PR or CLI fallback (`git-hosts.md`); fails → STOP. Restore `<previous-branch>` when testing ends.

### 5. Build the Test Plan

From `<coverage-map>`, write `<test-plan>` with four scenario classes:

- **Source** — one scenario per `<client-cases>` entry, replaying the client's case as reported: their steps, their data shapes, their flow. A client bug report → the exact reported failure must now pass; a client-specific feature request → the client's use case performed the way they described it. These are the primary scenarios — Delivery generalizes from them, never substitutes; an `<expected-outcomes>` entry a source scenario already covers needs no duplicate.
- **Delivery** — one user-level scenario per `<expected-outcomes>` entry: entry point, user action, expected observable result (UI state, DB effect, or both).
- **Data-artifact** — one execution scenario per migration, seeder, operation, or script flagged in Step 2, run by the user with three beats: the **pre-check** (inspect the provisioned pre-state — the rows or schema the artifact should change), the **command** (the exact invocation per project tooling), and the **post-check** (what must land — schema shape, rows, effects). The agent verifies landed effects in the DB — command output text is not evidence — plus any user-visible change a later scenario confirms.
- **Regression** — the surface past the diff: direct callers and shared modules traced via code navigation (`code-navigation.md`), each asserting unchanged behavior; project tests covering the changed symbols, run by the agent via file/name filter; and a **system pass**: the core journeys every user hits (auth, landing view, primary CRUD flow) exercised even where the diff never touches them.

Write each user-facing scenario so the user never guesses: numbered actions in the real UI (exact routes, navigation, field values, clicks), the expected observable result, the DB effect the agent will verify, and the data prerequisites Step 6 must provision. Order setup-free flows first.

**Done when:** every `<client-cases>` entry maps to a source scenario · every `<expected-outcomes>` entry maps to a delivery scenario · every flagged artifact maps to an execution entry · regression callers enumerated · system-pass journeys picked.

### 6. Provision Test Data

Treat missing local data as setup work. For each scenario, identify its required records, tenant context, relationships, permissions, and reproduction-critical values; inspect the live local DB to find what is missing.

Build the smallest faithful fixture: use existing generic seeds/factories where suitable; otherwise create the missing records or adjust dedicated test records through application tooling or direct local DB inserts/updates. Preserve the reported failure's relevant data shapes and relationships, including edge-case or legacy states that ordinary factories omit. Production copies are an option only for source scenarios or when local construction cannot reproduce the required state; unavailable production access alone does not block a locally constructible scenario.

Local provisioning is part of setup, not a repository fix: create fixtures in the local DB without editing tracked files. Additive local writes need no further permission; ask before overwriting or deleting existing local data. Prepare prerequisites only: the user exercises the real flow or flagged artifact in Step 7, rather than the agent writing its expected result into the DB.

Leave every flagged data artifact **pending**: the DB sits in the pre-artifact state the user inspects before running it. Provision the records each artifact acts on — the buggy rows an operation fixes, the data a migration transforms — on the same local dev DB the user will exercise. An artifact already applied locally → roll it back or re-provision so the user can run it themselves.

Production copy rules:

- Follow project database-tool and data-handling rules (`database-tools.md`). Confirm the source is production and the destination is local before importing.
- Copy only the records needed for the scenario, including required relationships and tenant context. Preserve reproduction-critical values; mask unrelated sensitive data and exclude secrets from logs, reports, and version control.
- Isolate local mail, queues, webhooks, and external integrations before exercising copied data so reproduction cannot affect real users or services.
- Write only to local — never to production. Ask before overwriting or deleting existing local data.

If provisioning fails, try the applicable alternatives above. Mark a scenario `blocked` only when a specific unmet prerequisite remains after concrete provisioning attempts, or a safety/permission boundary prevents them. Record the missing prerequisite, attempted methods and their errors or limitations, and what would unblock the scenario. Continue with scenarios whose prerequisites are available.

**Done when:** every scenario's required records exist locally and inspection confirms the reproduction-critical values and relationships, or that scenario has an evidenced provisioning blocker recorded for the report · every flagged artifact left pending with its pre-state provisioned, or `blocked` with evidence.

### 7. Guide the User Through the Test

Run the app per project tooling. Present the plan overview first — the scenario list and what it covers — so the user can pace the session, then walk one scenario at a time.

Per scenario: restate what setup is already in place, then hand the user the numbered actions (exact routes, clicks, inputs — or the exact command for a data artifact, with its pre-check and post-check) and the expected result. Wait for the user's report before advancing; use the interactive question tool for pass/fail/observation where one exists.

After each scenario, verify **persisted effects** via read-only DB inspection (`database-tools.md`): rows created/updated, status transitions, relationship integrity. The user's observation and the DB must agree — a mismatch is a failure; report it immediately with the evidence on both sides.

- A reported failure → capture evidence (what the user saw, error text, screenshot), guide one retry if the cause looks like a missed step, then record `fail` — debugging is a fix workflow, not this skill.
- A scenario the user cannot perform (for example, an unavailable device or external session) → `blocked` with evidence — never `pass` unverified. Missing local accounts or data return to Step 6; data-related blockers must meet its provisioning criteria.
- Reset or re-provision data between runs when the user needs to repeat a scenario.
- Run agent-side regression checks (diff-related tests, lint) alongside the guided pass; a test failure the diff did not cause is still a finding — mark it pre-existing after a cheap check against `<base-branch>`.

Record `<results>` per scenario: `pass`/`fail`/`blocked` with the user's observation plus agent DB evidence.

**Done when:** every scenario attempted · each `<expected-outcomes>` entry has a result · every artifact run by the user or `blocked` with reason · user observation and DB state reconciled per scenario · related-test results and system pass captured.

### 8. Report Findings

Chat only — never post to the PR. Per `<expected-outcomes>` entry: verified / failed / untested (with reason). Then:

- **Issues** — severity, reproduction steps, evidence, `file:line` where the diff is implicated.
- **Coverage gaps** — linked tickets/cards the PR does not fully address (`<coverage-map>` flags + failed scenarios + `<premise-drift>` items — assumed bug path or behavior gone, what superseded it).
- **Data artifacts** — what the user ran and what landed, or why `blocked`.
- **Regressions** — adjacent behavior that broke: regression scenarios, test failures (pre-existing vs caused), system-pass failures.
- **Verdict** — `works` only when delivered outcomes pass AND the system still works around them; otherwise `issues found`, one line.

## Notes

- The agent never performs the test actions — UI flows, migrations, operations, seeders, and scripts all belong to the user. If the agent can drive the UI itself, invoke `pr-test` instead.
- Verifies, never fixes — issues found → user invokes a fix workflow; leave the repo as found (no edits, no stray commits).
- An artifact too destructive to run locally (drops tables, mutates shared infra) → `blocked` with the risk named; review its code and report what verification the deploy needs instead.
