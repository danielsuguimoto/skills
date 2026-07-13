---
name: pr-fix
description: "Activate when a pull request has review feedback that requires code fixes and direct thread responses."
---

Address PR feedback by making fixes and replying directly to review threads — NEVER add standalone PR comments.

## Quick Reference

- Execution modes: `review` (default; ask before commit/push) or `auto` (commits/pushes without approval). Tools: git host tool (see `/docs/git-hosts.md` in the project root). Prefer MCP; fall back to CLI.
- ABSOLUTE RULE: Reply directly to review threads — never standalone PR comments. Follow `/docs/git-hosts.md` in the project root.

Delegate Steps 2, 4, and 5 to a subagent. Inline only on `BLOCKED`, `NO_PR_REF`, or `NO_FEEDBACK`. The parent owns Steps 1, 3, and 6-13.

> CRITICAL: Default mode is `review` — you MUST get explicit user approval (Step 9) before committing, pushing, or replying to threads. Never infer approval.

### 1. Interpret Arguments

- `auto` request → `<execution-mode>` = `auto`; PR number/URL → `<pr-ref>`; extra guidance/constraints → `<additional-context>`; otherwise → `<execution-mode>` = `review`.

> The user's `<pr-ref>` is the only authoritative source for the target PR. Never assume the current branch determines the target. If `<pr-ref>` points to a different branch, check it out in Step 3.

### 2. Delegate PR Analysis (primary path)

Resolve the project root via `git rev-parse --show-toplevel` (see `/docs/git-hosts.md` in the project root) → `<repo-path>`. Use it as `cwd` for every MCP call. Spawn a subagent with `<pr-ref>`, `<repo-path>`, and `<additional-context>`. Store the returned fenced markdown block as `<pr-analysis>`. Extract `<pr-number>`, `<pr-branch>`, and `<base-branch>` from the PR_NUMBER/PR_BRANCH/BASE_BRANCH sections. Run `git branch --show-current` → `<current-branch>`. Skip to Step 3. If the subagent is unavailable or returns `BLOCKED`, `MISSING_INPUT`, or `NO_PR_REF`, fall through to inline Steps 2, 4, and 5. If it returns `NO_FEEDBACK`, output "No changes".

### 2i. Load PR Context (inline fallback)

Primary thread source: git host list unresolved review comments (see `/docs/git-hosts.md` in the project root). It filters `isResolved=false` server-side, so resolved-thread bodies never enter parent context. CLI fallback per `/docs/git-hosts.md`: GraphQL `reviewThreads` with `isResolved` filter. Use git host list review comments/REST `pulls/{n}/comments` only as a `databaseId` cross-reference — never as the primary thread source, because it returns resolved comments too.

PR metadata: git host view PR (see `/docs/git-hosts.md` in the project root, fields: `number,title,body,url,headRefName,baseRefName,author,state,commits`) and git host list reviews (see `/docs/git-hosts.md` in the project root). CLI fallback per `/docs/git-hosts.md`:
```bash
gh pr view <pr-number> --json number,title,body,url,headRefName,baseRefName,author,state,commits
gh api repos/{owner}/{repo}/pulls/<pr-number>/reviews
```

Extract `<pr-branch>`, `<base-branch>`, `<pr-number>`, and `<pr-url>`. Run `git branch --show-current` → `<current-branch>`. Review attachments (images, PDFs, linked files) when relevant. Note gaps if inaccessible.

### 3. Align Local Branch

If `<pr-branch>` is unavailable, STOP. Otherwise check out via git host checkout PR (see `/docs/git-hosts.md` in the project root) or CLI `gh pr checkout <pr-number>` per `/docs/git-hosts.md`. Store the active branch as `<active-branch>`. If checkout fails, STOP. Don't modify code until `<active-branch>` == `<pr-branch>`.

> Never skip checkout even if workspace looks related. `<pr-ref>` is authoritative.

### 4i. Load Changes (inline fallback)

Use git host PR diff (see `/docs/git-hosts.md` in the project root). CLI fallback per `/docs/git-hosts.md`: `gh pr diff <pr-number>` or `git diff <base-branch>...<active-branch>`. Store as `<changes>`.

### 5i. Analyze Feedback (inline fallback)

1. Review open, unresolved review threads; check review states (`CHANGES_REQUESTED`, `APPROVED`, etc.)
2. Use `<changes>` to understand the current diff. Prioritize critical issues (bugs, security, broken contracts). Identify files needing changes.
3. Effectively resolved threads (skip, don't re-fix or re-reply): not marked resolved in GitHub, but have a reply stating addressed ("Fixed...", "Done...") and a later commit implementing the fix (verify via commit diff). If in doubt, ask the user. Threads already resolved by GitHub are excluded upstream in Step 2i; this step catches only the GitHub-unresolved subset.
4. For each actionable thread, record `<comment-id>`, `<file>`, `<lines>`, `<category>`, `<suggestion>`, `<reviewer-body>`, and `<current-heading>`. Don't blindly follow every suggestion — some lead off course.

### 5p. Parse Delegated Analysis (primary path)

From `<pr-analysis>`, build the actionable-thread list. For each thread in the THREADS section with category `critical`, `important`, or actionable `style`, record `<comment-id>`, `<file>`, `<lines>`, `<category>`, `<suggestion>` (one-line), and `<current-heading>`. Do not persist `<reviewer-body>` in parent context for actionable threads — pass it through the group-assignment payload to subagents in Step 7. Carry SUMMARY's recommended fix order and GROUPS partition into Step 6. Threads marked `noise` or `question` go to the `not-fixing` list with reasons; record `<reviewer-body>` only there (needed for Step 12 reply justification).

### 6. Plan Implementation

Fast path: if every actionable thread is a straightforward, specific code suggestion with no conflicts, skip planning. Set `<implementation-plan>` = `trivial` and go to Step 7 serial path. Skip the fast path when any thread is ambiguous, has cross-thread dependencies, or needs judgment. Otherwise synthesize actionable threads into an internal plan (no approval gate):

1. Build an ordered fix list: sort by priority (`critical` → `important` → `style`), then file, then line. Move effectively resolved, `noise`, and `current-heading: yes` threads to a separate `not-fixing` list with reasons.
2. Evaluate validity. Mark threads `invalid` (misreading, would break behavior, contradicts PR intent) with a one-line reason and add them to the `not-fixing` list. No fix is needed, but reply explaining why in Step 12. Optional parallel investigation: spawn background subagents to investigate validity only when ≥4 actionable threads need deep code lookups and subagent spawning is available. Partition into batches of 3-5. Subagents return `valid`/`invalid` + reason + evidence; don't let them make cross-thread validity calls — flag those for the parent. Skip when threads are simple, few, or surface-level.
3. Map dependencies between threads (shared imports, types, callers) → `<dependencies>`.
4. Identify file-disjoint groups. Consume the GROUPS partition from `<pr-analysis>` when present. Override only when cross-thread deps invalidate the analyzer's partition; otherwise don't re-derive. Cross-boundary threads stay in the parent's serial pass.
5. Draft subagent task inputs per group → `<group-assignments>`.
6. Execution path: parallel (≥2 file-disjoint groups, no cross-group deps) or serial (single thread, shared files, or subagent spawning unavailable).
7. Store `<implementation-plan>`: ordered fix list, `not-fixing` list, `<dependencies>`, `<group-assignments>`, and execution path.

### 7. Implement Fixes

Parallel path: for each group, spawn a background subagent with group inputs. The parent handles cross-file or ambiguous threads serially. Await results via `read_subagent` and merge them. Handle `blocked-cross-file` in the parent. Serial path: fix critical issues first, follow existing patterns, use `<active-branch>`, make focused minimal changes, and be ready to explain why you maintained your heading. After either path, store `<changes-count>` and collect `<subagent-validation>`.

### 8. Validate Changes

- Delegate lint and tests to a foreground subagent: project lint and test commands (see project config or `/docs/` in the project root) on the changed files. If the subagent is unavailable, run the same commands inline.
- Confirm the fixes address the feedback. Store `<validation-results>` and `<validation-passing>` (`yes`/`no`).

### 9. Review Fixes With User — MANDATORY APPROVAL GATE

Never skip. Get explicit approval before commit, push, or thread replies. Don't infer approval from silence or "looks good". Only `decision: approved` counts. If `<validation-passing>` = `no`, STOP before any commit, push, or reply. If `<execution-mode>` = `auto`, skip the gate and go to Step 11. Otherwise the gate is mandatory.

9a. Present gate: show the fix summary, changed file count, and validation results. Ask one question (header `Approval`): "Ready to commit, push, and respond on the PR with these changes?" Options: `Go Ahead` (accept without modifications) or `Needs Review` (requires feedback explaining what to change).

9b. Normalize:
| Selection | Side text | `decision` | `feedback` |
|-----------|-----------|------------|------------|
| `Go Ahead` | — | `approved` | `""` |
| `Needs Review` | empty | invalid — re-prompt | |
| `Needs Review` | non-empty | `concerns` | custom_text |

9c. Route:
| `decision` | Action |
|------------|--------|
| `approved` | Exit loop → Step 11 |
| `concerns` + feedback | Store `<review-feedback>` → Step 10 → re-implement → re-validate → re-present gate |

Approval loop (mandatory): `Needs Review` does not exit or authorize anything. Loop: apply feedback (Step 10) → re-implement (Step 7) → re-validate (Step 8) → re-present gate (9a). Repeat until `approved`. No other exit. When `<changes-count>` = 0, surface the no-change state so the user picks `Go Ahead` (no-op) or `Needs Review` with feedback.

### 10. Apply Review Feedback

Use `<review-feedback>` to refine without widening scope unless explicitly asked. Return to Step 7, re-validate in Step 8, and re-present gate in Step 9. No commit, push, or reply during this step.

### 11. Commit And Push — ONLY AFTER `approved`

Run only after the approval loop exits with `approved`, or when `<execution-mode>` = `auto`. Stage (`git add -A`), commit (`git commit -m "<commit-message>"`), and push (`git push origin <active-branch>`). Follow git conventions for the commit message — never use a generic message like "address review feedback". Store `<pushed>` as `yes` or `no`.

### 12. Respond to Threads

Reply to addressed threads only after commit and push succeed (and were authorized).

CRITICAL — NEVER VIOLATE:
- ALWAYS reply to specific review comment threads via thread reply API
- NEVER add standalone PR review comments (`gh pr review -c` or `gh pr comment`)
- Every response = direct reply to existing review thread

Sole exception — flat PR comments: if the reviewer left a flat PR comment (no line anchor), post a single flat PR comment that quote-replies (blockquote original + author handle, then response). This is the only permitted use of git host PR comment (see `/docs/git-hosts.md` in the project root) or `gh pr comment` (CLI) per `/docs/git-hosts.md`. Correct: use git host reply to review thread (see `/docs/git-hosts.md` in the project root, `pr_number`, `comment_id`, `body`). CLI fallback per `/docs/git-hosts.md`:

```bash
gh api repos/{owner}/{repo}/pulls/<pr-number>/comments/<comment-id>/replies \
  --method POST --input - <<'EOF'
{"body":"<reply-text>"}
EOF
```

Avoid `-f body=` with special characters (use `--input -`), `gh pr review -c`, and `gh pr comment` (standalone). Keep replies short and factual. Confirm what was addressed and what was intentionally not followed.

### 13. Resolve Addressed Threads

After replying to fully addressed threads, resolve them via GraphQL `resolveReviewThread` (GraphQL-only; no REST endpoint). Use git host list review threads (see `/docs/git-hosts.md` in the project root) to map comment IDs to thread node IDs (`PRRT_...`), then git host resolve review thread (see `/docs/git-hosts.md` in the project root, `thread_id`).

CLI fallback:
```bash
# List unresolved threads
gh api graphql -f query='{ repository(owner: "<owner>", name: "<repo>") { pullRequest(number: <pr-number>) { reviewThreads(first: 100) { nodes { id isResolved comments(first: 1) { nodes { databaseId body author { login } } } } } } } }'

# Resolve
gh api graphql -f query='mutation { resolveReviewThread(input: {threadId: "<thread-node-id>"}) { thread { isResolved } } }'
```

Match by the first comment's `databaseId` (equals REST comment ID). Rules: only resolve threads where feedback was fully addressed. Don't resolve threads you intentionally didn't follow (leave them open with reasoning). Don't resolve threads the reviewer reopened. Store the count as `<threads-resolved>`.

## Output

Output a summary of changes made and validation results.

- At the approval gate: one line per fix applied, not full diffs. Example: "Fixed N threads across M files" + validation status.

