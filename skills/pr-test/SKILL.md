---
name: pr-test
description: "Test or verify a PR through real entry points, with the user or agent driving. Supports `quick` and `budget <N>`."
---

### 0. Confirm the Test Driver

Before loading PR context, checking out a branch, or preparing runtime/data, ask: **Who will drive the test actions?** Use the interactive question tool where available, with these choices:

- **User** — the user performs test actions; the agent guides and verifies them.
- **Agent** — the agent performs the test actions through real entry points and verifies effects; ask for human-only access or actions when needed.

Record the answer as `<test-driver>` = `user` or `agent`. An unanswered or skipped choice pauses the workflow; there is no default driver. Keep the choice for the session and change it only on the user's request. For `user`, read and follow [user-driven testing](user-driven.md); both drivers use the same scope, plan, preparation, and proof rules below.

## Required `<project-root>/docs` reads

Shell-read before testing the PR; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`; `<project-root>/docs/issue-trackers.md` for linked requirements · `<project-root>/docs/database-tools.md` when provisioning or verifying data · `<project-root>/docs/code-navigation.md` only when locating an in-scope entry point or diagnosing a concrete failure · `<project-root>/docs/browser-automation.md` for agent-driven UI execution.

Act as a human QA engineer. Spend most effort proving `<pr-purpose>` through realistic use. Test the main journeys and clear, critical edge cases; touched files and methods are not a coverage checklist. Stop when the scoped behavior has evidence, not when every nearby code path has been exercised.

**Boundary:** local runtime and database changes follow the project's safety and approval rules. Confirm the database target before migrations, seeders, operations, or fixture writes; obtain required approval. Keep production read-only and leave the PR thread, application source/tests, and repository history untouched. Local file writes are limited to the ignored QA report package and an explicitly approved narrow repository-local exclusion under [reporting.md](reporting.md). Ask before overwriting or deleting existing local data. Testing runs against the existing local database, destructive artifacts included.

**Completion gate:** every planned scenario has a result. `PASSED` means execution met its assertions; `FAILED` means it contradicted an assertion; `BLOCKED` means a prerequisite could not be prepared or a required action was unavailable/declined under Step 6, with the missing prerequisite, attempted preparation and recovery or reason none was suitable, and unblock path recorded; `DEFERRED` means requested depth, scenario budget, or the user ending the session excluded it, with a reason. Missing evidence never means `PASSED` or `FAILED`.

### 1. Interpret Arguments

PR number/URL → `<pr-ref>`; `quick` → `<test-depth>` = `quick` (default `full`); `budget <N>` → `<scenario-budget>` = `<N>` (default uncapped); focus areas or constraints → `<additional-context>`. No PR reference → derive from conversation or the current branch's open PR (`git-hosts.md`).

### 2. Load PR Context

Read the PR and diff (`git-hosts.md`). Extract `<pr-purpose>`, `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`, and the tickets that define its requirements as `<ticket-refs>`. Distinguish those tickets from historical or incidental references. Unloadable PR → STOP.

Identify `<data-artifacts>` — shipped migrations, seeders, operations, and scripts — and any clear, critical risk directly caused by the change. Read code only to locate execution entry points, understand a reproduction-critical condition, or diagnose a concrete failure. Shared code changes alone do not require caller inventories or an application-wide pass.

### 3. Load Linked Context

Load each `<ticket-refs>` entry's body and checklists (`issue-trackers.md`). Read comments, attachments, or nested links only to resolve a missing requirement or reproduction detail; this scope also governs generic tracker loading instructions. Tool failure → try a configured alternative. If user access or missing content is necessary, ask once; unavailable → record the gap and continue with supported requirements.

Extract `<expected-outcomes>` as the observable behavior the PR promises. Implementation steps, helper methods, test-writing tasks, and validation commands are not separate user journeys; use them to locate behavior or existing checks. Record `<client-cases>` when the source contains a customer report, keeping the reported steps and data conditions that matter to the failure. Historical context does not expand the PR's purpose.

Build a compact `<coverage-map>` from these outcomes to assertions in the planned journeys or focused checks. Flag missing implementation. An outcome whose premise no longer exists → `<premise-drift>`; explain it in the report rather than inventing a scenario for it.

**Done when:** every defining ticket is loaded or has a recorded gap; every observable promised outcome and reported customer case from loaded sources is recorded; shipped artifacts and identified critical risks are listed; the coverage map records any missing implementation and premise drift.

### 4. Align Local Branch

Record `<previous-branch>`. Uncommitted changes → STOP, ask the user to stash/commit. Check out the PR through project tooling (`git-hosts.md`); failure → STOP. Restore `<previous-branch>` when testing ends.

### 5. Build the Test Plan

Write a small `<test-plan>` around natural user journeys. A journey may span pages, actors, and lifecycle steps. Admit a new scenario only when it proves a distinct in-scope behavior or critical risk that selected execution evidence does not already prove. Put related assertions into the same journey; scenario labels do not require separate executions.

- **Purpose** — exercise the PR's promised behavior, including the reported customer failure where applicable. Reuse that execution for the requirements it proves.
- **Data-artifact** — in `full`, execute every `<data-artifacts>` entry against representative populated local data and verify its schema/row effects. Fold this into a purpose scenario when the same execution proves both; otherwise add a dedicated artifact scenario. Re-run only where idempotency is claimed.
- **Critical edge** — add only an explicit requirement or an obvious, high-impact failure tied to the purpose or a changed security/data-integrity boundary. State the concrete risk. Use representative cases, not permutations of roles, tenants, dates, flags, and legacy states. Keep detailed input matrices in existing focused tests.

`quick` selects the main-purpose journeys, including the reported failure, and defers standalone artifact and optional edge checks. Run and verify artifacts that are prerequisites for those journeys, recording them within the journey rather than deferring required setup. A failed `quick` scenario stays `FAILED`; expand depth only when the user requests it.

Prioritize the purpose, then clear critical risks. Select complete dependency chains before applying `<scenario-budget>`: prerequisites run before dependent assertions and count within a journey when they are part of the same scenario. If a required chain cannot fit, defer it and select a smaller complete journey rather than a check whose prerequisite was cut. Apply depth and budget before detailed setup or data provisioning.

Give each scenario an ID, source outcome, entry point, prerequisites, and observable assertions; add the risk for an edge scenario. Read [reporting.md](reporting.md) before materializing this plan; assign its scenario/assertion IDs and intended proof checkpoints without adding scenarios. Deferred IDs need only their source and reason. Out-of-scope permutations need no scenario rows; summarize that boundary once. Every non-drifted promised outcome maps to a selected assertion or a deferred ID; implementation and automated-test checklists do not inflate this ledger.

**Done when:** the plan proves the purpose, accounts for required artifacts and clear critical risks, has executable dependency chains, and adds no scenario whose evidence another selected execution already supplies. Initialize its in-progress report under reporting.md before Step 6; unresolved storage safety or replacement approval pauses testing.

### 6. Prepare Minimal Local Data

Prepare the main journey first and later prerequisites when needed. Start only the required local processes and bring the local database to a working baseline — apply the project's migrations and seeders where it ships them, except selected `<data-artifacts>`; prepare their pre-state, then execute them under the selected driver through Step 7's pre-state/execution/post-state procedure before preparing any dependent journey. Inspect existing local data and reuse a suitable tenant, account, and records. Create missing scenario data and relationships through the real UI under the selected driver, existing project seeds/factories, or targeted application or local database commands. Try applicable paths until the required state exists; an empty database or missing records alone is not a blocker. Prepare only what the selected behavior needs; preserve reproduction-critical values without recreating the customer's whole environment.

Use existing tooling and small, one-off commands. Do not create setup scripts, seeders, fixture generators, verification harnesses, or bulk fixture matrices for this QA pass. Reuse compatible records across a journey. Setup supplies prerequisites, never the behavior's expected result; the selected driver executes shipped artifacts as tests in Step 7, not as hidden fixture setup.

For an already-applied artifact, reconstruct a representative pre-state and make the artifact runnable through its real entry point. Create affected records in their original state. If schema or execution tracking prevents a rerun, restore only the affected pre-state using a project-supported rollback or targeted local commands; the selected driver runs any shipped artifact involved. Confirm the local target and obtain required approval before rollback, overwriting, deletion, or destructive schema changes; do not reset, clone, or re-provision the database merely to obtain a clean run. Inspect the reconstructed pre-state, execute the artifact, and verify its effects under Step 7. An already-run artifact or already-repaired data alone is not a blocker, and a no-op against repaired data is not proof of the original change.

Request required access or approval before treating its absence as a blocker. Record a scenario `BLOCKED` only when its required state or execution cannot be obtained through applicable preparation paths within the safety rules, or required access or approval is unavailable or declined. Record the paths attempted, the specific missing capability or approval, and the unblock path; continue independent journeys.

Production copies are an option only when a purpose scenario needs historical state that lightweight local preparation cannot reproduce. Follow `database-tools.md`: confirm production source and local destination; copy only needed records and relationships; preserve reproduction-critical values while masking unrelated sensitive data; isolate mail, queues, webhooks, and external integrations before use. Keep secrets out of evidence and version control; write only to local.

**Recovery:** group setup/tool failures by missing prerequisite. Capture the error, diagnose the cause, and make at most one simple local repair-and-retry for that failure across the run. If it fails, try other applicable preparation paths before marking dependent scenarios `BLOCKED` with the recovery record and unblock path. Reopen when new evidence or user-supplied access resolves the blocker. Diagnose application failures only enough to record the finding, not to fix the repo or investigate unrelated code. Record setup errors and dependent blockers in the report, reusing shared proof rather than creating separate unindexed logs.

For external dependencies, use an available sandbox or existing controlled local substitute at the external boundary while executing the real application path. Record the substitution and proof limit; a substitute does not prove provider acceptance. If a criterion requires real provider execution and access is unavailable, ask once for the needed action, then record `BLOCKED` if unavailable.

**Done when:** the next selected scenario has the minimum faithful local prerequisites or a recorded blocker. Deferred scenarios need no data preparation.

### 7. Exercise or Guide the Change

Run coherent journeys serially with shared runtime, browser session, and records. Parallel execution is optional only for genuinely independent agent-driven scenarios with compatible setup under the `subagents` gate; shared mutable state stays serial, and parallelism does not justify duplicate fixture preparation.

With `<test-driver>` = `agent`, perform the selected actions and collect the results. With `<test-driver>` = `user`, follow `user-driven.md`.

The selected driver exercises UI journeys as a regular user through real navigation, forms, and clicks; agent-driven UI uses `browser-automation.md`. Exercise API contracts, workers/jobs, CLI commands, and artifacts through their real entry points. Console shortcuts, API calls, and DB writes do not replace the main user-facing journey.

For each selected shipped artifact, inspect the pre-state, have the selected driver run the exact command through project tooling in dependency order, then inspect schema and affected rows. Preserve existing rows where required, verify intended changes, and capture any claimed idempotency rerun. Reuse that execution for its purpose and artifact assertions; command success alone does not prove data correctness.

Prove each required persistence or data-integrity outcome with the minimum sufficient evidence. Reuse the journey or an executed existing test when it proves that outcome; a success message or optimistic UI alone does not prove persistence. Query (`database-tools.md`) only when that evidence is insufficient, selecting the relevant records and fields rather than auditing every write or dumping tables. Record the finding and reuse its proof; UI-only assertions need no query. Keep the selected shipped-artifact pre/post checks above. An asserted persisted effect that is wrong fails even if the UI looks right. For repeats, reuse valid existing state and prepare only what the repeat needs within safety and approval rules; no automatic database rebuild.

Use existing focused tests to answer in-scope questions the QA journeys have not proved; select checks by the unanswered risk, not by every touched file. An executed integration test can prove the backend assertions and real entry points it exercises; unit tests prove their narrower calculations. Reuse this evidence for covered critical cases rather than replaying their matrix manually. Neither proves unexercised browser behavior or an unexecuted shipped artifact. Keep the main QA journey and the runtime evidence appropriate to its entry point.

Capture and organize proof checkpoints under [reporting.md](reporting.md), updating the report as execution proceeds. Reuse one artifact for every assertion or scenario it proves rather than repeating execution, screenshots, or queries for the same observed state.

Record `<results>` by ID: steps/command, expected versus observed behavior, evidence paths, and status; user-driven actions include the user's report. An application error reached through the real scenario is `FAILED`; a setup/tool error preventing it follows Step 6. Label a failure pre-existing only with evidence; otherwise leave attribution unconfirmed instead of expanding into an unrelated investigation.

Reconcile the scoped outcomes, plan, and results once: each selected assertion has evidence or a recorded blocker/defer reason, required capture gaps are explicit, each deferred ID has a reason, and planned count = `PASSED` + `FAILED` + `BLOCKED` + `DEFERRED`. Apply reporting.md's artifact/link integrity checks; fill gaps within the selected scope, keep deferred work deferred and unchanged blockers blocked, and stop when this gate passes.

### 8. Report Findings

Finalize the local HTML report under [reporting.md](reporting.md); never post findings to the PR.

- **Verdict** — name the driver. `FAILED` if an evidenced executed scenario failed; otherwise `INCOMPLETE` if no purpose journey executed, a blocker remains, a defining ticket is missing, a required capture has a proof gap, or mixed revisions prevent a clean tested-revision claim; otherwise `PASSED` for the exercised scope. A report delivery blocker prevents an unqualified pass. Include passed, failed, blocked, and deferred counts.
- **Report** — give the absolute `index.html` location. Keep scenario details, expected versus observed behavior, evidence, failures, and test boundaries in the report rather than repeating its table in chat.
- **Actionable findings** — summarize concrete failures, blockers/unblock paths, missing requirements or defining tickets, premise drift, and proof gaps that need attention. Cite `file:line` only where the diff is implicated; distinguish confirmed pre-existing failures from unknown attribution. Claim only what the evidence proves, not exhaustive application coverage.

## Notes

Verifies, never fixes. Leave application source/tests and repository history as found: no code or test additions, setup scripts, stray commits, or PR updates. Retain only the authorized ignored report package and any explicitly approved narrow local exclusion; fixes require a separate request.
