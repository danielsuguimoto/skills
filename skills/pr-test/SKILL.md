---
name: pr-test
description: "Use when testing or verifying a pull request end-to-end — exercise the change as a regular user (UI + database), verify every linked ticket/card is delivered, and catch regressions. READ-ONLY; reports findings in chat. Often run after the PR is approved."
---

## Required `<project-root>/docs` reads

Read these project-root spec files before testing the PR (use shell `cat`/`ls` — they may be in `.gitignore`, invisible to built-in search). Missing file → fall back to native tools, note the gap; never invent contents.

- `<project-root>/docs/browser-automation.md`
- `<project-root>/docs/code-navigation.md`
- `<project-root>/docs/database-tools.md`
- `<project-root>/docs/git-hosts.md`
- `<project-root>/docs/issue-trackers.md`

Test whether a PR **works**, not whether it reads well. Load the PR's purpose and linked tickets/specs, check out the branch, drive the system as a regular user, verify persisted effects in the database, confirm coverage of every related card, and probe for regressions. READ-ONLY — no code changes, no commits, nothing posted to the PR. Findings go to chat.

### 1. Interpret Arguments

- PR number/URL → `<pr-ref>`; extra focus areas or constraints → `<additional-context>`; no arguments → derive from conversation or the open PR for the current branch (git host list PRs, see `<project-root>/docs/git-hosts.md` in the project root).

### 2. Load PR Context

View the PR via git host view PR + PR diff (see `<project-root>/docs/git-hosts.md`). Extract `<pr-purpose>` (why it was created — title, body, commit messages), `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`, `<changes>`, and `<ticket-refs>` (every issue/card reference in the body, commits, or branch name — on the git host or any external tracker). Unloadable PR → STOP.

### 3. Load Linked Context

For each `<ticket-refs>` entry, load the ticket via the issue tracker tool (see `<project-root>/docs/issue-trackers.md`): body, every comment, checklists (e.g. Implementation/Validation sections), attachments.

**Follow nested links.** A PR commonly links to a git host issue that itself links to a card on another platform — the real requirements live on the inner card. After loading each ticket, scan its body, comments, checklists, and attachments for references to other issues/cards on any platform. Load each new reference, repeat the scan on it, and add it to `<ticket-refs>` until no new refs surface. Dedupe visited refs so cross-linked tickets don't loop. Every nested ticket contributes to `<expected-outcomes>`.

Extract `<expected-outcomes>` — one entry per acceptance criterion, validation item, or spec requirement the PR must deliver. A linked ticket that fails to load → record a gap, don't guess its criteria.

Build `<coverage-map>`: map each `<expected-outcomes>` entry to the diff files/modules that should deliver it. Flag outcomes with no corresponding change — the PR may not address every linked card.

**Done when:** every linked ticket loaded or gapped · every acceptance criterion/validation item in `<expected-outcomes>` · coverage map built with gaps flagged.

### 4. Align Local Branch

Record `<previous-branch>` via `git branch --show-current`. Uncommitted changes → STOP, ask user to stash/commit. Check out via git host checkout PR or CLI fallback per `<project-root>/docs/git-hosts.md`. Checkout fails → STOP. Restore `<previous-branch>` when testing ends.

### 5. Build the Test Plan

From `<coverage-map>`, write `<test-plan>`: one user-level scenario per `<expected-outcomes>` entry — entry point, user action, expected observable result (UI state, DB effect, or both). Add the regression surface: callers and shared modules the diff touches beyond its own feature, traced via code navigation (see `<project-root>/docs/code-navigation.md`) — each becomes a scenario asserting unchanged behavior. Include the project's own test suite when configured. Order scenarios so setup-free flows run first; note test-data prerequisites per scenario.

### 6. Exercise the Change

Run the app per the project's tooling. Drive each `<test-plan>` scenario through browser automation (see `<project-root>/docs/browser-automation.md`) as a **regular user** — real UI navigation, forms, clicks. No console shortcuts, no direct API calls, no DB writes to force state; seed/fixture setup only when a scenario genuinely requires it.

After each scenario, verify **persisted effects** via read-only database inspection (see `<project-root>/docs/database-tools.md`): rows created/updated, status transitions, relationship integrity. The live database is ground truth — a UI that looks right while the DB is wrong is a failure.

Regressions: run the project's test/lint commands where they exist, plus the regression-surface scenarios from Step 5.

Record `<results>` per scenario: `pass` / `fail` / `blocked`, with evidence (observed UI state, query results, error text).

**Done when:** every scenario attempted · each `<expected-outcomes>` entry has a result · regression surface exercised · suite results captured.

### 7. Report Findings

Chat only — never post to the PR. Per `<expected-outcomes>` entry: verified / failed / untested (with reason). Then:

- **Issues** — severity, reproduction steps, evidence, `file:line` where the diff is implicated.
- **Coverage gaps** — linked tickets/cards the PR does not fully address (from `<coverage-map>` flags and failed scenarios).
- **Regressions** — adjacent behavior that broke.
- **Verdict** — `works` or `issues found`, one line.

## Notes

- This skill verifies; it never fixes. Issues found → user invokes a fix workflow separately.
- Leave the repo as found: no code edits, no stray commits.
- A scenario that cannot be exercised locally (external service, missing seed) → mark `blocked` with the reason, never mark `pass` unverified.
