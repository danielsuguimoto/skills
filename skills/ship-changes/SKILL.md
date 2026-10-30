---
name: ship-changes
description: "Use when shipping current work end-to-end — creating a branch, committing, pushing, and opening a PR. Invoke when the user says 'ship this', 'create a PR', or when work is complete and ready for review."
---

## Required `<project-root>/docs` reads

Read via shell before shipping end-to-end — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/git-hosts.md`
- `<project-root>/docs/issue-trackers.md`

## Operations

| Operation | Tool | CLI fallback |
|-----------|--------------|--------------|
| Pre-scan change state | git host change scan | `git branch --show-current` + `git status --short` + `git diff --stat` + `git diff --cached --stat` + `git log @{u}..HEAD --oneline` |

All git/gh operations follow `<project-root>/docs/git-hosts.md`. Forward git host tool availability to subskills.

### 1. Interpret Arguments

Ticket URL/reference → `<ticket-url>` · wording influencing branch name/commit → `<additional-context>` · base branch → `<base>`.

### 2. Pre-Scan Changes (Shared Context)

Run the change scan once per the table → `<current-branch>`, `<uncommitted>` (status + diff stat), `<ahead-commits>` (`@{u}..HEAD` log, empty if none).

Build `<change-summary>` from `<uncommitted>` (primary) or `<ahead-commits>` (fallback): 2-4 logical themes, what and why (not how), considering `<additional-context>`.

Derive `<branch-category>` from `<change-summary>` (`feature`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `build`, `ci`).

Blocker: `<uncommitted>` and `<ahead-commits>` both empty → STOP: "Nothing to ship."

Forward to every subskill: `<current-branch>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<branch-category>` — subskills then skip their own Load Changes, Analyze Scope, and Determine Category steps.

### 3. Create Work Branch

Skip when `<current-branch>` is NOT the detected default (`git remote show origin | grep "HEAD branch" | awk '{print $NF}'`) AND is already ticket-related → `<new-branch>` = `<current-branch>`.

Otherwise invoke `new-branch`, passing `<additional-context>`, `<current-branch>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<branch-category>` → `<new-branch>`. "Nothing to branch from" or "Branching skipped" → STOP and report.

### 4. Commit and Push

`<uncommitted>` empty → skip the commit substep; still push when `<ahead-commits>` is non-empty.

Invoke `commit-and-push`, passing `<additional-context>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>` → `<hash>`, `<push-target>`. "Nothing to commit or push" → STOP and report.

### 5. Create Pull Request

Create the PR, passing `<change-summary>` (reused — scan post-commit git state but keep the semantic summary), `<ticket-url>` when defined, `<base>` when defined. Output the PR URL. Verify the pr-create subskill posted the `<pr-url>` mention in every referenced ticket (its Step 10); skipped → run it now.

**Mandatory:** the pr-create subskill must run its PR-template check (its Step 6) before composing the body — forwarded `<change-summary>` triggers its Step-4 skip path but does NOT exempt the template check. Accept `<pr-template>` = `none` only when the subskill confirms it probed the repo.
