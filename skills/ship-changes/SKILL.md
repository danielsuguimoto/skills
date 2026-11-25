---
name: ship-changes
description: "Ship current work end-to-end: branch, commit, push, and open a PR. Use for explicit shipping requests, not a PR-only request."
---

Run the change scan, branch, commit, push, PR creation, and ticket linking in the current session, in order. Invoke subskills inline: load their instructions and execute them here. Delegate only the read-only Ticket Brief unless the user explicitly asks to delegate another step.

An explicit shipping request authorizes branch creation, commit, push, and PR creation. Execute them without asking for separate confirmation; honor mandatory safety gates.

## Required `<project-root>/docs` reads

Shell-read before shipping end-to-end; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`

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

Invoke `commit-and-push`, passing `<additional-context>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<excluded>` → `<hash>`, `<push-target>`. "Nothing to commit or push" → STOP and report.

### 5. Create Pull Request

Invoke `pr-create` with the inputs above. Rescan post-commit git state but keep the semantic `<change-summary>`. Output the PR URL. Verify the pr-create subskill posted the `<pr-url>` mention in every referenced ticket (its Step 10); skipped → run it now.

**Mandatory:** the pr-create subskill must run its PR-template check (its Step 6) before composing the body — forwarded `<change-summary>` triggers its Step-4 skip path but does NOT exempt the template check. Accept `<pr-template>` = `none` only when the subskill confirms it probed the repo.

## Ticket Brief

Reuse a current session brief, summary, or ticket content covering the references and scoped answers, with sufficient evidence and decision status for that scope. Otherwise delegate one isolated, read-only subagent: pass the ticket references, scope, `<project-root>`, these rules, and the target PR reference when known; the parent receives only the brief.

Worker: read `<project-root>/docs/issue-trackers.md` and use its read operations. Start with each ticket's title, body, checklists, and requested metadata; dedupe by tracker and ID. When scope includes intended, expected, or current behavior or constraints, check the latest page of ticket comments for updates. Read older comments, attachment contents, or linked tickets only when needed to answer the scope or resolve conflicting or unclear decisions. These scoped rules override the spec's general attachment and nested-link traversal. Do not treat the target PR as a ticket or fetch its comments or reviews; return scope-relevant target PR decision references as evidence gaps for the parent. Follow other PR links only when needed for the scope, and retrieve specific relevant evidence rather than full comment or review history. If a provider bundles extra content, inspect only what the scope requires.

Return a compact cited brief with scoped answers, decision status, blockers, and evidence gaps; cite ticket, comment permalink, or attachment name and location as appropriate. Omit raw threads and attachment contents. Distinguish confirmed decisions from proposals and superseded decisions when evidenced; mark answers partial when evidence indicates uninspected or inaccessible material could change them. Stop when the scoped answers are supported; never post or modify tracker state. Worker or spec unavailable → report the gap without fetching inline.
