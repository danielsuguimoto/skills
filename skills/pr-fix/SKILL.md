---
name: pr-fix
description: "Activate when a pull request has review feedback that requires code fixes and direct thread responses."
---

## Required `<project-root>/docs` reads

Shell-read before fixing PR review feedback; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`

Fix PR feedback and reply in review threads. Tools: git host (`git-hosts.md`).

Modes: `review` (default) or `auto` — `auto` skips the Step 9 approval gate.

### 1. Interpret Arguments
`auto` → `<execution-mode>` = `auto`; PR number/URL → `<pr-ref>`; extra guidance → `<additional-context>`; otherwise → `review`.

### 2. Load PR Context
`git rev-parse --show-toplevel` → `<repo-path>` (use as `cwd` for MCP calls). Primary thread source: git host list unresolved review comments (`isResolved=false` server-side). CLI fallback: GraphQL `reviewThreads` with `isResolved`. REST `pulls/{n}/comments` only as `databaseId` cross-reference — never primary (returns resolved too).

Non-thread feedback (collect every run — threads alone miss it): git host list reviews → non-empty review bodies; git host list flat comments. Store both as `<flat-feedback>` (author, body, comment id). Skip bot-authored and own-agent comments.

PR metadata: git host view PR + list reviews (CLI fallback per spec). Extract `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`. `git branch --show-current` → `<current-branch>`. Review attachments when relevant. No threads and no `<flat-feedback>` → output "No changes".

### 3. Align Local Branch
`<pr-branch>` unavailable → STOP. Check out via git host checkout PR or `gh pr checkout <pr-number>` → `<active-branch>`. Fails → STOP. No code edits until `<active-branch>` == `<pr-branch>`. Never skip checkout — `<pr-ref>` is authoritative.

### 4. Load Changes
Git host PR diff. Fallback: `gh pr diff <pr-number>` or `git diff <base-branch>...<active-branch>` → `<changes>`.

### 5. Analyze Feedback
1. Review unresolved threads; check review states (`CHANGES_REQUESTED`, `APPROVED`, etc.). Every `<flat-feedback>` item is reviewer feedback, not metadata.
2. Use `<changes>` to understand the diff. Prioritize critical issues (bugs, security, broken contracts).
3. Effectively resolved threads (skip): GitHub-unresolved but have "Fixed..."/"Done..." reply + later commit implementing it — verify via commit diff. In doubt → ask.
4. Per actionable thread record `<comment-id>`, `<file>`, `<lines>`, `<category>`, `<suggestion>`, `<reviewer-body>`, `<current-heading>`; flat items the same minus `<file>`/`<lines>`. Evaluate each suggestion against `<changes>` before applying. `noise`/`question` threads → `not-fixing` list with reasons (`<reviewer-body>` recorded there only, for Step 12).

### 6. Plan Implementation
Fast path: every thread is a straightforward suggestion with no conflicts → `<implementation-plan>` = `trivial`, skip to Step 7. Skip fast path when any thread is ambiguous, has cross-thread dependencies, or needs judgment. Otherwise: ordered fix list (`critical` → `important` → `style`, then file, then line); effectively-resolved/`noise`/`current-heading: yes` → `not-fixing`; `invalid` threads (misreading, would break behavior, contradicts PR intent) → `not-fixing` with reason (reply why in Step 12). Dependencies → `<dependencies>`.

### 7. Implement Fixes
Critical first, existing patterns, `<active-branch>`, focused minimal changes → `<changes-count>`.

### 8. Validate Changes
Lint + tests on changed files. Confirm fixes address feedback → `<validation-results>`, `<validation-passing>` (`yes`/`no`).

### 9. Approval Gate (mandatory in `review` mode)
Never skip — only `decision: approved` counts; don't infer from silence. `<validation-passing>` = `no` → STOP. `auto` mode → skip to Step 11.

Present gate: fix summary, changed file count, validation results. Ask (header `Approval`): "Ready to commit, push, and respond?" Options: `Go Ahead` → `approved`; `Needs Review` → `concerns` (requires feedback).

- `approved` → Step 11. `concerns` → `<review-feedback>` → Step 10 → re-implement (7) → re-validate (8) → re-present. Repeat until `approved`.
- `<changes-count>` = 0 → surface no-change state so user picks `Go Ahead` (no-op) or `Needs Review` with feedback.

### 10. Apply Review Feedback
Use `<review-feedback>` to refine without widening scope → Step 7. No commit/push/reply during this step.

### 11. Commit and Push (after `approved` or `auto`)
`git add -A`, `git commit -m "<message>"`, `git push origin <active-branch>`. Never generic messages like "address review feedback" → `<pushed>` (`yes`/`no`).

### 12. Respond to Threads
Reply only after commit + push succeed. ALWAYS reply via thread reply API — NEVER standalone PR comments (`gh pr review -c`, `gh pr comment`). Sole exception: flat PR comments (no line anchor) → one flat comment quote-replying (blockquote original + author handle, then response).

Git host reply to review thread (`pr_number`, `comment_id`, `body`; fallback per spec — `--input -` for special chars). Replies short and factual.

### 13. Resolve Addressed Threads
After replying, resolve fully addressed threads via GraphQL `resolveReviewThread` (GraphQL-only; no REST). Map comment IDs to thread node IDs (`PRRT_...`) by first-comment `databaseId` via git host list review threads (fallback per spec), then git host resolve review thread. Only fully addressed threads — never intentionally skipped or reviewer-reopened ones. Flat comments have no resolution — reply only (Step 12) → `<threads-resolved>`.

## Output
Summary of changes and validation results. At the approval gate: one line per fix, not full diffs — e.g. "Fixed N threads across M files" + validation status.
