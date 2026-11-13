---
name: ship-changes
description: "Use when shipping current work end-to-end — branch, commit, push, open PR — e.g. 'ship this', 'create a PR', or work complete and ready for review."
---

## Required `<project-root>/docs` reads

Shell-read before shipping end-to-end; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`, `<project-root>/docs/issue-trackers.md` (Step 2 only — `<ticket-url>` provided)

## Operations

| Operation | Tool | CLI fallback |
|-----------|--------------|--------------|
| Pre-scan change state | git host change scan | change-scan fallback per spec |

All git/gh operations follow `git-hosts.md`. Forward git host tool availability to subskills.

### 1. Interpret Arguments

Ticket URL/reference → `<ticket-url>` · wording influencing branch name/commit → `<additional-context>` · base branch → `<base>`.

### 2. Pre-Scan Changes (Shared Context)

Run the change scan once per the table → `<current-branch>`, `<uncommitted>` (status + file list + diff stat), `<ahead-commits>` (`@{u}..HEAD` log, empty if none). If the change scan omits file paths, collect the full changed-file list (untracked included) via the change-scan fallback (`git-hosts.md`).

Anchor `<ship-intent>` — why the changes exist: the session's work and, when `<ticket-url>` is defined, the ticket's title/summary via the issue tracker tool (`issue-trackers.md`). Tool fails → fall back to session work and diff; note the gap. Diff stats and commit messages carry the what; intent supplies the why.

Reconcile `<uncommitted>` against `<ship-intent>`: each changed file maps to a theme or is flagged. Inspect the file's diff (per `git-hosts.md`) when its role is unclear. Flagged files (unrelated work, debug artifacts, secrets/dotenv) — ask the user; never sweep them in. Store as `<excluded>` (empty when none).

Derive `<included>` = changed files in `<uncommitted>` that are not in `<excluded>`.

Blocker: `<included>` empty and `<ahead-commits>` empty → STOP: "Nothing to ship."

Build `<change-summary>` from `<included>` (primary) or `<ahead-commits>` (fallback), grounded in `<ship-intent>`: 2-4 logical themes, what and why (not how), considering `<additional-context>`.

Derive `<branch-category>` from `<change-summary>` per `new-branch` categories.

Forward to subskills:
- `new-branch`: `<additional-context>`, `<current-branch>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<branch-category>`.
- `commit-and-push`: `<additional-context>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<excluded>`.
- `pr-create`: `<change-summary>`, `<excluded>`, `<ticket-url>` when defined, `<base>` when defined.

### 3. Create Work Branch

Skip when `<current-branch>` is NOT the detected default (per `git-hosts.md` fallback) AND is already ticket-related → `<new-branch>` = `<current-branch>`.

Otherwise invoke `new-branch`, passing `<additional-context>`, `<current-branch>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<branch-category>` → `<new-branch>`. "Nothing to branch from" or "Branching skipped" → STOP and report.

### 4. Commit and Push

`<included>` empty → skip the commit substep; still push when `<ahead-commits>` is non-empty.

Invoke `commit-and-push`, passing `<additional-context>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<excluded>` → `<hash>`, `<push-target>`. "Nothing to commit or push" → STOP and report.

### 5. Create Pull Request

Create the PR, passing `<change-summary>` (reused — scan post-commit git state but keep the semantic summary), `<excluded>`, `<ticket-url>` when defined, `<base>` when defined. Output the PR URL. Verify the pr-create subskill posted the `<pr-url>` mention in every referenced ticket (its Step 10); skipped → run it now.

**Mandatory:** the pr-create subskill must run its PR-template check (its Step 6) before composing the body — forwarded `<change-summary>` triggers its Step-4 skip path but does NOT exempt the template check. Accept `<pr-template>` = `none` only when the subskill confirms it probed the repo.
