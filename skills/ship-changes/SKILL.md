---
name: ship-changes
description: "Ship current work end-to-end — branch, commit, push, open a PR — for explicit shipping requests, not a PR-only request."
---

Run the change scan, branch, commit, push, PR creation, and ticket linking in the current session, in order. Invoke selected installed subskills inline; use [standalone.md](standalone.md) for the fallback paths below. A Ticket Brief may run inline or be delegated when the delegation gates permit; delegate another step only on explicit request.

An explicit shipping request authorizes branch creation, commit, push, and PR creation. Execute them without asking for separate confirmation; honor mandatory safety gates.

## Required `<project-root>/docs` reads

Shell-read before shipping end-to-end (reuse session reads): `<project-root>/docs/git-hosts.md`. If a spec or operation is missing, report the gap and read [the bundled fallback spec](docs/git-hosts.md).

## Operations

| Operation | Tool | CLI fallback |
|-----------|--------------|--------------|
| Pre-scan change state | git host change scan | change-scan fallback per spec |

All git/gh operations follow `git-hosts.md`. Forward git host tool availability to subskills.

### 1. Interpret Arguments

Explicit ticket URL/reference → `<ticket-url>` · explicit skip → `<ticket-mode>` = `skip` · wording influencing branch name/commit → `<additional-context>` · base branch → `<base>`. Without an explicit ticket or skip, use the unambiguous ticket targeted by the session's current work; ambiguous → ask; no session ticket → `<ticket-mode>` = `skip`. A selected ticket → `<ticket-mode>` = `provided`; `skip` → `<ticket-url>` empty.

### 2. Pre-Scan Changes (Shared Context)

Run the change scan once per the table → `<current-branch>`, `<uncommitted>` (status + file list + diff stat), `<ahead-commits>` (`@{u}..HEAD` log — no upstream, e.g. detached `HEAD` → `origin/<default>..HEAD`; empty if none). If the scan omits paths or collapses untracked directories, collect individual changed file paths via the [bundled scan](docs/git-hosts.md), including every untracked file.

When `<ticket-url>` is defined, get a ticket brief (see Ticket Brief) for the ticket title and the problem the changes address. Store the ticket reference and brief or blocker as `<ticket-brief-result>`, even if no usable brief returns. Anchor `<ship-intent>` in the session's work and the brief; no usable brief → fall back to session work and diff, and note the gap. Diff stats and commit messages carry the what; intent supplies the why.

Reconcile `<uncommitted>` against `<ship-intent>`: each changed file maps to a theme or is flagged. Inspect the file's diff (per `git-hosts.md`) when its role is unclear. Flagged files (unrelated work, debug artifacts, secrets/dotenv) — ask the user; never sweep them in. Store as `<excluded>` (empty when none).

Derive `<included>` = changed files in `<uncommitted>` that are not in `<excluded>`.

Blocker: `<included>` empty and `<ahead-commits>` empty → STOP: "Nothing to ship."

Build `<change-summary>` from `<included>` (primary) or `<ahead-commits>` (fallback), grounded in `<ship-intent>`: 2-4 logical themes, what and why (not how), considering `<additional-context>`.

Derive `<branch-category>` from `<change-summary>` per [standalone.md](standalone.md#branch).

If either `pr-create` or `commit-and-push` is unavailable, resolve the push destination before publication per [standalone.md](standalone.md#destination). If `pr-create` is unavailable, also preflight its standalone PR-creation path before branching or publishing. A destination that cannot be determined or represented → stop before publication. No configured host operation and no usable documented fallback → stop and report.

Forward to subskills:
- `new-branch`: `<additional-context>`, `<current-branch>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<branch-category>`, `<ticket-url>`, `<ticket-mode>`, `<ticket-brief-result>`.
- `commit-and-push`: `<additional-context>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<excluded>`.
- `pr-create`: `<change-summary>`, `<excluded>`, `<ticket-url>`, `<ticket-mode>`, `<ticket-brief-result>`, and `<base>` when defined.

### 3. Create Work Branch

Skip when `<current-branch>` is NOT the detected default (per `git-hosts.md` fallback) AND is already ticket-related → `<new-branch>` = `<current-branch>`.

Otherwise invoke `new-branch`, passing `<additional-context>`, `<current-branch>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<branch-category>`, `<ticket-url>`, `<ticket-mode>`, `<ticket-brief-result>` → `<new-branch>`; if unavailable, follow [standalone.md](standalone.md#branch). "Nothing to branch from" or "Branching skipped" → STOP and report.

### 4. Commit and Push

`<included>` empty → skip the commit substep; still push when `<ahead-commits>` is non-empty.

Refresh the index. Any staged `<excluded>` path → stop before committing or pushing; ask the user to preserve or unstage it. Leave the index untouched.

When `pr-create` is unavailable or `commit-and-push` is unavailable, follow [standalone.md](standalone.md#commit-and-push) with the verified destination; the installed publisher has no destination input. Otherwise invoke `commit-and-push`, passing `<additional-context>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<excluded>` → `<hash>`, `<push-target>`. Set `<published-head>` = `<hash>`. "Nothing to commit or push" → STOP and report.

### 5. Create Pull Request

Invoke `pr-create` with the inputs above plus `<published-head>` and `<push-target>`; if unavailable, follow [standalone.md](standalone.md#pull-request). Rescan post-commit git state but keep the semantic `<change-summary>`. Output the PR URL. Verify the PR link was posted to every verified, in-scope ticket; if the link step was skipped, run it once now — no duplicate retries.

**Mandatory:** probe the PR template before composing the body, including on the standalone path. Accept `<pr-template>` = `none` only after the probe completes.

## Ticket Brief

Reuse current, scoped evidence; otherwise use `issue-trackers.md` to dedupe by tracker/ID, read only scoped fields, and read the latest comment page for behavior or constraints. Follow older material or other links only to settle a gap; for the target PR return decision references without fetching feedback. Return cited answers, decision status (confirmed/proposed/superseded), gaps and blockers; mark partial answers and leave tracker state unchanged. Read inline, or delegate one bounded read-only brief when it saves distracting context or the user requests it; pass the refs, scope, `<project-root>` and target PR. Worker unavailable → inline; completed lookup blocker → do not retry. Missing spec → report the gap.
