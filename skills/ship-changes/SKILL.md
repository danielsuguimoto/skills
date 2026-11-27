---
name: ship-changes
description: "Ship current work end-to-end — branch, commit, push, open a PR — for explicit shipping requests, not a PR-only request."
---

Run the change scan, branch, commit, push, PR creation, and ticket linking in the current session, in order. Invoke installed subskills inline; use the standalone steps below when a companion is missing. A Ticket Brief may run inline or be delegated when the delegation gates permit; delegate another step only on explicit request.

An explicit shipping request authorizes branch creation, commit, push, and PR creation. Execute them without asking for separate confirmation; honor mandatory safety gates.

## Required `<project-root>/docs` reads

Shell-read before shipping end-to-end (reuse session reads): `<project-root>/docs/git-hosts.md`. Missing spec or operation → report the gap; use available standard Git and host interfaces, or stop if unavailable.

## Operations

| Operation | Tool | CLI fallback |
|-----------|--------------|--------------|
| Pre-scan change state | git host change scan | change-scan fallback per spec |

All git/gh operations follow `git-hosts.md`. Forward git host tool availability to subskills.

### 1. Interpret Arguments

Explicit ticket URL/reference → `<ticket-url>` · explicit skip → `<ticket-mode>` = `skip` · wording influencing branch name/commit → `<additional-context>` · base branch → `<base>`. Without an explicit ticket or skip, use the unambiguous ticket targeted by the session's current work; ambiguous → ask; no session ticket → `<ticket-mode>` = `skip`. A selected ticket → `<ticket-mode>` = `provided`; `skip` → `<ticket-url>` empty.

### 2. Pre-Scan Changes (Shared Context)

Run the change scan once per the table → `<current-branch>`, `<uncommitted>` (status + file list + diff stat), `<ahead-commits>` (`@{u}..HEAD` log — no upstream, e.g. detached `HEAD` → `origin/<default>..HEAD`; empty if none). Collect individual changed file paths, including untracked files, when the scan omits paths or collapses directories.

When `<ticket-url>` is defined, get a ticket brief (see Ticket Brief) for the ticket title and the problem the changes address. Store the ticket reference and brief or blocker as `<ticket-brief-result>`, even if no usable brief returns. Anchor `<ship-intent>` in the session's work and the brief; no usable brief → fall back to session work and diff, and note the gap. Diff stats and commit messages carry the what; intent supplies the why.

Reconcile `<uncommitted>` against `<ship-intent>`: each changed file maps to a theme or is flagged. Inspect the file's diff (per `git-hosts.md`) when its role is unclear. Flagged files (unrelated work, debug artifacts, secrets/dotenv) — ask the user; never sweep them in. Store as `<excluded>` (empty when none).

Derive `<included>` = changed files in `<uncommitted>` that are not in `<excluded>`.

Blocker: `<included>` empty and `<ahead-commits>` empty → STOP: "Nothing to ship."

Build `<change-summary>` from `<included>` (primary) or `<ahead-commits>` (fallback), grounded in `<ship-intent>`: 2-4 logical themes, what and why (not how), considering `<additional-context>`.

Derive `<branch-category>` from `<change-summary>`: `feature`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `build`, or `ci`.

If either `pr-create` or `commit-and-push` is unavailable, use the standalone publisher and PR path together. Before branching, resolve the base host/repository and push remote/repository; ambiguous → ask. Verify host support for the head, including cross-fork ownership; unsupported or unavailable → stop. Resolve the full destination ref before publishing.

Forward to subskills:
- `new-branch`: `<additional-context>`, `<current-branch>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<branch-category>`, `<ticket-url>`, `<ticket-mode>`, `<ticket-brief-result>`.
- `commit-and-push`: `<additional-context>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<excluded>`.
- `pr-create`: `<change-summary>`, `<excluded>`, `<ticket-url>`, `<ticket-mode>`, `<ticket-brief-result>`, and `<base>` when defined.

### 3. Create Work Branch

Skip when `<current-branch>` is NOT the detected default (per `git-hosts.md` fallback) AND is already ticket-related → `<new-branch>` = `<current-branch>`.

Otherwise invoke `new-branch`, passing `<additional-context>`, `<current-branch>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<branch-category>`, `<ticket-url>`, `<ticket-mode>`, `<ticket-brief-result>` → `<new-branch>`. Missing companion → create `<branch-category>/<slug>` at the current checkout without tracking the base; slug is 2–4 kebab-case words from the ticket title or summary. Existing work branch → skip; name collision → try `-2` once. "Nothing to branch from" or "Branching skipped" → STOP and report.

### 4. Commit and Push

`<included>` empty → skip the commit substep; still push when `<ahead-commits>` is non-empty.

Refresh the index. Any staged `<excluded>` path → stop before committing or pushing; ask the user to preserve or unstage it. Leave the index untouched.

Both companions installed → invoke `commit-and-push`, passing `<additional-context>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<excluded>` → `<hash>`, `<push-target>`.

Standalone → stage and commit only `<included>` paths; message `<prefix>: <message>`, lowercase, under 50 characters. Skip the commit when `<included>` is empty. Record the verified remote ref's tip, then push to that exact remote and ref; set upstream only when absent, preserving existing tracking. Rejected non-fast-forward → fetch that ref, replay only commits above the recorded tip, and retry once; missing tip, merge commits, conflict, or other failure → stop; abort a conflicting rebase. Never force-push. Return HEAD as `<hash>` and the actual remote and full ref as `<push-target>`.

Set `<published-head>` = `<hash>`. "Nothing to commit or push" → STOP and report.

### 5. Create Pull Request

Both companions installed → invoke `pr-create` with the inputs above plus `<published-head>` and `<push-target>`.

Standalone → select the requested or default base, fetch it from the verified repository, and compare its three-dot diff with HEAD. Rescan individual untracked files; stop for changes outside `<excluded>`, staged exclusions, a base-branch checkout, or an empty diff. Probe PR templates; fill the selected template without changing its structure, or use Ticket, Issue, Description, Checklist. Include the problem, changes, and ticket or `SKIPPED`; title follows the commit convention. Verify `<push-target>` matches the preflight target and its remote ref matches HEAD. Create or find the PR against the verified base and head, pinning the host/repository for lookup; verify the rendered body. Link the PR in each verified, in-scope ticket; ask on ambiguous targets and report failed links.

Rescan post-commit git state but keep the semantic `<change-summary>`. Output the PR URL. Verify the PR link was posted to every verified, in-scope ticket; if the link step was skipped, run it once now — no duplicate retries.

**Mandatory:** probe the PR template before composing the body. Accept `<pr-template>` = `none` only after the probe completes.

## Ticket Brief

Reuse current, scoped evidence; otherwise use `issue-trackers.md` to dedupe by tracker/ID, read only scoped fields, and read the latest comment page for behavior or constraints. Follow older material or other links only to settle a gap; for the target PR return decision references without fetching feedback. Return cited answers, decision status (confirmed/proposed/superseded), gaps and blockers; mark partial answers and leave tracker state unchanged. Read inline, or delegate one bounded read-only brief when it saves distracting context or the user requests it; pass the refs, scope, `<project-root>` and target PR. Worker unavailable → inline; completed lookup blocker → do not retry. Missing spec → report the gap.
