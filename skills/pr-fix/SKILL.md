---
name: pr-fix
description: "Activate when a pull request has review feedback requiring code fixes and replies — inline threads, review bodies, or flat comments."
---

## Required `<project-root>/docs` reads

Shell-read before fixing PR review feedback; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`

Fix PR feedback and reply in review threads. Tools: git host (`git-hosts.md`).

Modes: `review` (default) or `auto` — `auto` skips the Step 9 approval gate.

### 1. Interpret Arguments
`auto` → `<execution-mode>` = `auto`; PR number/URL → `<pr-ref>`; extra guidance → `<additional-context>`; otherwise → `review`.

### 2. Load PR Context
`git rev-parse --show-toplevel` → `<repo-path>` (use as `cwd` for MCP calls). Primary thread source: git host list unresolved review comments (`isResolved=false` server-side). CLI fallback: GraphQL `reviewThreads` with `isResolved`. REST `pulls/{n}/comments` only as `databaseId` cross-reference — never primary (returns resolved too).

Non-thread feedback (collect every run — threads alone miss it): git host list reviews → non-empty review bodies; git host list flat comments. Store both as `<flat-feedback>` (author, body, comment id). Review bodies are reviewer feedback regardless of author — review bots report findings there. A finding restated across a reviewer's bodies counts once — keep the latest restatement. Skip own-agent items and automation notices (CI, coverage, deploys).

PR metadata: git host view PR + list reviews (CLI fallback per spec). Extract `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`. `git branch --show-current` → `<current-branch>`. Review attachments when relevant. No threads and no `<flat-feedback>` → output "No changes".

Anchor `<pr-intent>` — why the PR exists: title, body, commit messages, and a ticket brief (see Ticket Brief) for all issue/card references in the body, commits, or branch name, scoped to intended behavior and constraints. No usable brief → fall back to PR title/body; note the gap. Threads supply the what; intent arbitrates scope.

### 3. Align Local Branch
`<pr-branch>` unavailable → STOP. Inspect the worktree and index; uncommitted or untracked changes → STOP and ask the user to preserve them before continuing. Fetch the PR head and record its commit → `<pr-head-commit>`. Check out via git host checkout PR or `gh pr checkout <pr-number>` → `<active-branch>`. Fails → STOP. Behind `<pr-head-commit>` → sync it yourself per `git-hosts.md`: fast-forward to `<pr-head-commit>` — behind alone never blocks. Diverged or unpushed local commits → no safe automatic update → STOP and report. No code edits until `<active-branch>` == `<pr-branch>` and HEAD == `<pr-head-commit>`. Never skip checkout — `<pr-ref>` is authoritative.

### 4. Load Changes
Git host PR diff. Fallback: `gh pr diff <pr-number>` or `git diff <base-branch>...<active-branch>` → `<changes>`.

### 5. Analyze Feedback
1. Review unresolved threads; check review states (`CHANGES_REQUESTED`, `APPROVED`, etc.). Every `<flat-feedback>` item is reviewer feedback, not metadata.
2. Use `<changes>` to understand the diff. Prioritize critical issues (bugs, security, broken contracts).
3. Effectively resolved (skip): threads with a "Fixed..."/"Done..." reply plus a commit newer than the finding implementing it, or flat items addressed by a commit newer than the item or confirmed fixed in a later reviewer body — verify via commit diff. In doubt → ask.
4. Per actionable thread record `<comment-id>`, `<file>`, `<lines>`, `<category>`, `<suggestion>`, `<reviewer-body>`, `<current-heading>`; flat items the same minus `<file>`/`<lines>`. Evaluate each suggestion against `<changes>` and `<pr-intent>` before applying. `noise`/`question` items → `not-fixing` list with reasons (`<reviewer-body>` recorded there only, for Step 12).
5. Premise drift — a suggestion or `<pr-intent>` assumes a bug path, invariant, or behavior the diff/code no longer has (superseding change, misdiagnosed cause) → `not-fixing` candidate; ambiguous → ask. Never fix against a dead premise.
6. Scope creep — a suggestion is sound but outside `<pr-intent>` (pre-existing bug, adjacent refactor, broader coverage, "while you're here" work) → `not-fixing` candidate; ambiguous → ask. Belongs to a separate change, not this PR.

### 6. Plan Implementation
Fast path: every actionable item is a straightforward suggestion with no conflicts → `<implementation-plan>` = `trivial` — on this path partition items into `<fix-groups>` (shared file lands in one group) and mark `parallel-eligible` per the criteria below before skipping to Step 7 when dispatch is viable: three or more groups or the user's explicit request. Skip fast path when any item is ambiguous, has cross-item dependencies, or needs judgment. Otherwise: ordered fix list (`critical` → `important` → `style`, then file, then line); effectively-resolved/`noise`/`current-heading: yes` → `not-fixing`; `invalid` items (misreading, would break behavior, contradicts `<pr-intent>`) and `out-of-scope` items (sound but outside it) → `not-fixing` with reason (reply why in Step 12). Map cross-item dependencies as sequential thinking → `<dependencies>`; partition actionable items into `<fix-groups>` and mark a group `parallel-eligible` when its files are disjoint from the other groups, it has no cross-group dependencies, and it runs no commands writing shared mutable state such as lockfiles or build caches.

### 7. Implement Fixes
Critical first, existing patterns, `<active-branch>`, focused minimal changes — apply each item's ask, no adjacent refactor, hardening, or cleanup. `parallel-eligible` `<fix-groups>` may be implemented in parallel — dispatch one subagent per group, each editing only its assigned files — only with three or more groups or the user's explicit request to delegate, and only when a suitable subagent profile exists; doubtful independence stays serial. Collect each group's changed files into `<fix-files>`, verifying they stayed within its boundaries; a failed group's changes are discarded or reset before validating or committing — never mix successful and failed group edits; its items are retried serially in this session, and items still unfixed are surfaced at the approval gate, never counted as fixed. Validation, commit, and thread replies stay serial in this session. → `<changes-count>` and `<fix-files>`.

### 8. Validate Changes
Lint + tests on changed files. Confirm fixes address feedback → `<validation-results>`, `<validation-passing>` (`yes`/`no`).

### 9. Approval Gate (mandatory in `review` mode)
Never skip — only `decision: approved` counts; don't infer from silence. `<validation-passing>` = `no` → STOP. `auto` mode → skip to Step 11.

Present gate: fix summary, changed file count, validation results. Ask (header `Approval`): "Ready to commit, push, and respond?" Options: `Go Ahead` → `approved`; `Needs Review` → `concerns` (requires feedback).

- `approved` → Step 11. `concerns` → `<review-feedback>` → Step 10 → re-implement (7) → re-validate (8) → re-present. Repeat until `approved`.
- `<changes-count>` = 0 → surface no-change state so user picks `Go Ahead` (no-op) or `Needs Review` with feedback.

### 10. Apply Review Feedback
Use `<review-feedback>` to refine without widening scope → Step 7. No commit/push/reply during this step. Re-derive `<fix-groups>` and `parallel-eligible` marks when `<review-feedback>` changes which files a fix touches — stale boundaries invalidate the prior partition.

### 11. Commit and Push (after `approved` or `auto`)
Inspect the worktree and index again, including changes within `<fix-files>`. Unexpected changes → STOP and preserve them. Stage and commit only `<fix-files>` with explicit pathspecs, then `git push origin <active-branch>`. Never use a generic message like "address review feedback" → `<pushed>` (`yes`/`no`).

### 12. Respond to Threads
Reply after commit + push succeed — with nothing pushed (`<changes-count>` = 0), reply after gate approval instead; reviewers are still owed the `not-fixing` reasons. ALWAYS reply via thread reply API — NEVER standalone PR comments (`gh pr review -c`, `gh pr comment`). Sole exception: flat items (flat comments, review bodies — no line anchor) → one flat comment per item quote-replying (blockquote original + author handle, then response).

Git host reply to review thread (`pr_number`, `comment_id`, `body`; fallback per spec — `--input -` for special chars). Replies short and factual.

### 13. Resolve Addressed Threads
After replying, resolve fully addressed threads via GraphQL `resolveReviewThread` (GraphQL-only; no REST). Map comment IDs to thread node IDs (`PRRT_...`) by first-comment `databaseId` via git host list review threads (fallback per spec), then git host resolve review thread. Only fully addressed threads — never intentionally skipped or reviewer-reopened ones. Flat items have no resolution — reply only (Step 12) → `<threads-resolved>`.

## Ticket Brief

When a step calls for a ticket brief, reuse session results first — a prior brief covering the same references and scope, or the referenced tickets' content already loaded at the depth the brief requires (comments and attachments included; a lighter read does not qualify) → derive the scoped answers without delegating. Otherwise delegate to one isolated, read-only subagent: pass only the ticket references, the scope, `<project-root>`, and these rules — the main session receives only the returned brief. Worker: read `<project-root>/docs/issue-trackers.md` and use its read operations; load each in-scope ticket's body, checklists, full comment thread, and accessible attachments (inspect contents, not filenames); follow only links bearing on the scope; dedupe by tracker and ID. Return a compact cited brief — the scoped answers, confirmed decisions distinguished from proposals and superseded ones, blockers and evidence gaps; cite comment permalinks or attachment name plus location; omit raw threads and attachment contents; mark the brief partial when missing evidence could change it. Never post or modify tracker state. Worker or spec unavailable → report the gap without fetching inline.

## Output
Summary of changes and validation results. At the approval gate: one line per fix, not full diffs — e.g. "Fixed N threads across M files" + validation status.
