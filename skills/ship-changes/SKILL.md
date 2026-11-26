---
name: ship-changes
description: "Ship current work end-to-end: branch, commit, push, and open a PR. Use for explicit shipping requests, not a PR-only request."
---

Run the change scan, branch, commit, push, PR creation, and ticket linking in the current session, in order. Invoke subskills inline: load their instructions and execute them here. A Ticket Brief may run inline or be delegated when the delegation gates permit; delegate another step only on explicit request.

An explicit shipping request authorizes branch creation, commit, push, and PR creation. Execute them without asking for separate confirmation; honor mandatory safety gates.

## Required `<project-root>/docs` reads

Shell-read before shipping end-to-end (reuse session reads): `<project-root>/docs/git-hosts.md`. If a spec is missing, report the gap and use a documented fallback.

## Operations

| Operation | Tool | CLI fallback |
|-----------|--------------|--------------|
| Pre-scan change state | git host change scan | change-scan fallback per spec |

All git/gh operations follow `git-hosts.md`. Forward git host tool availability to subskills.

### 1. Interpret Arguments

Explicit ticket URL/reference → `<ticket-url>` · explicit skip → `<ticket-mode>` = `skip` · wording influencing branch name/commit → `<additional-context>` · base branch → `<base>`. Without an explicit ticket or skip, use the unambiguous ticket targeted by the session's current work; ambiguous → ask; no session ticket → `<ticket-mode>` = `skip`. A selected ticket → `<ticket-mode>` = `provided`; `skip` → `<ticket-url>` empty.

### 2. Pre-Scan Changes (Shared Context)

Run the change scan once per the table → `<current-branch>`, `<uncommitted>` (status + file list + diff stat), `<ahead-commits>` (`@{u}..HEAD` log — no upstream, e.g. detached `HEAD` → `origin/<default>..HEAD`; empty if none). If the change scan omits file paths, collect the full changed-file list (untracked included) via the change-scan fallback (`git-hosts.md`).

When `<ticket-url>` is defined, get a ticket brief (see Ticket Brief) for the ticket title and the problem the changes address. Store the ticket reference and brief or blocker as `<ticket-brief-result>`, even if no usable brief returns. Anchor `<ship-intent>` in the session's work and the brief; no usable brief → fall back to session work and diff, and note the gap. Diff stats and commit messages carry the what; intent supplies the why.

Reconcile `<uncommitted>` against `<ship-intent>`: each changed file maps to a theme or is flagged. Inspect the file's diff (per `git-hosts.md`) when its role is unclear. Flagged files (unrelated work, debug artifacts, secrets/dotenv) — ask the user; never sweep them in. Store as `<excluded>` (empty when none).

Derive `<included>` = changed files in `<uncommitted>` that are not in `<excluded>`.

Blocker: `<included>` empty and `<ahead-commits>` empty → STOP: "Nothing to ship."

Build `<change-summary>` from `<included>` (primary) or `<ahead-commits>` (fallback), grounded in `<ship-intent>`: 2-4 logical themes, what and why (not how), considering `<additional-context>`.

Derive `<branch-category>` from `<change-summary>` per `new-branch` categories.

Forward to subskills:
- `new-branch`: `<additional-context>`, `<current-branch>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<branch-category>`, `<ticket-url>`, `<ticket-mode>`, `<ticket-brief-result>`.
- `commit-and-push`: `<additional-context>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<excluded>`.
- `pr-create`: `<change-summary>`, `<excluded>`, `<ticket-url>`, `<ticket-mode>`, `<ticket-brief-result>`, and `<base>` when defined.

### 3. Create Work Branch

Skip when `<current-branch>` is NOT the detected default (per `git-hosts.md` fallback) AND is already ticket-related → `<new-branch>` = `<current-branch>`.

Otherwise invoke `new-branch`, passing `<additional-context>`, `<current-branch>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<branch-category>`, `<ticket-url>`, `<ticket-mode>`, `<ticket-brief-result>` → `<new-branch>`. "Nothing to branch from" or "Branching skipped" → STOP and report.

### 4. Commit and Push

`<included>` empty → skip the commit substep; still push when `<ahead-commits>` is non-empty.

Invoke `commit-and-push`, passing `<additional-context>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<excluded>` → `<hash>`, `<push-target>`. Set `<published-head>` = `<hash>`. "Nothing to commit or push" → STOP and report.

### 5. Create Pull Request

Invoke `pr-create` with the inputs above plus `<published-head>` and `<push-target>`. Rescan post-commit git state but keep the semantic `<change-summary>`. Output the PR URL. Verify the pr-create subskill posted the `<pr-url>` mention in every referenced ticket (its Step 10); skipped → run it now.

**Mandatory:** the pr-create subskill must run its PR-template check (its Step 6) before composing the body — forwarded `<change-summary>` triggers its Step-4 skip path but does NOT exempt the template check. Accept `<pr-template>` = `none` only when the subskill confirms it probed the repo.

## Ticket Brief

Reuse current session evidence that covers the references, scoped answers, and decision status. Otherwise read inline, or delegate one isolated, read-only inspection when a bounded scan would add distracting context and a cited brief suffices, or the user requests delegation. Give the worker the references, scope, `<project-root>`, these rules, and target PR when known. Worker cannot run → read inline; spec unavailable → report the gap. An evidence blocker returned by a completed lookup is final, not a reason to retry.

Reader: read `<project-root>/docs/issue-trackers.md`. Dedupe by tracker and ID; read title and the body, checklists, or metadata needed for the scope. Read the latest comment page when scope includes behavior or constraints. Read older comments, attachments, and linked tickets only to close a scope gap or settle an unclear decision; this overrides general tracker traversal. For the target PR, return relevant decision references without fetching its feedback. Follow other PR links only for specific evidence the scope needs. Ignore unrelated bundled content; leave tracker state unchanged.

Return cited answers, decision status (confirmed, proposed, or superseded), blockers, and evidence gaps. Cite tickets, comment permalinks, or attachment names and locations; omit raw threads and attachments. Mark answers partial if unseen material could change them. Stop when the scope is supported.
