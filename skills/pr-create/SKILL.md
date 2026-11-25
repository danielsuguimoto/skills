---
name: pr-create
description: "Open a pull request for the current branch, standalone or during shipping."
---

## Required `<project-root>/docs` reads

Shell-read before creating the PR (reuse session reads): `<project-root>/docs/git-hosts.md`, `<project-root>/docs/issue-trackers.md` (Step 10 only — a ticket to link). If a spec is missing, report the gap and use a documented fallback.

## Operations

MCP tools take business-level inputs (title, body, base, head) — no CLI flags. All git host ops per `git-hosts.md` (detect branch+default base, check uncommitted, fetch base+diff, push, create issue, create/view PR, auth). Ticket comments via issue tracker update-ticket (`issue-trackers.md`).

As a subskill, the invoking skill owns Steps 1-10; on "Nothing to ship.", proceed to Step 3 blockers.

An explicit PR or shipping request authorizes the normal branch push and PR creation. A verified ticket selected for this workflow authorizes posting the PR link there as part of Step 10; create a ticket only in the explicitly selected `auto` mode. Do not ask again for these covered actions. Honor safety gates and tool-enforced permissions.

### 1. Interpret Arguments

Base branch → `<base>`; explicit auto-create ticket → `<ticket-mode>` = `auto`; explicit ticket URL/reference → `<ticket-url>` + `<ticket-mode>` = `provided`; explicit skip → `<ticket-mode>` = `skip`; guidance/focus areas → `<additional-context>`; files deliberately left uncommitted → `<excluded>`; no base → detect default in Step 2. Without an explicit ticket choice, retain a forwarded `<ticket-mode>`; otherwise use a forwarded `<ticket-url>` or the unambiguous ticket targeted by the session's current work as `provided`. Ambiguous → ask; no session ticket → `<ticket-mode>` = `skip`. `skip` → `<ticket-url>` = `SKIPPED`; `auto` → `<ticket-url>` empty until a ticket is created.

### 2. Load & Analyze Changes

- **Context.** `git branch --show-current` → `<current-branch>` (`git-hosts.md`). No `<base>` → detect default base per spec fallback. `<resolved-base>` = user `<base>` or detected default (strip `origin/`). Use `<remote-base>` = `origin/<resolved-base>` for all comparisons — the local base is often stale.
- **Changes.** `git status --short` + `git fetch origin <resolved-base>`; diff against `<remote-base>` with **three-dot syntax** (`...`) — mandatory, excludes base-branch-only changes: ahead log, `git diff --stat`, `git diff` → `<changes>`.

### 3. Check Blockers

Uncommitted changes outside `<excluded>` → STOP (report + list files). A staged `<excluded>` path → STOP (staged work escapes the exclusion). On base branch → STOP (suggest `git checkout -b <feature-name>`). No changes → STOP ("Nothing to include in a PR.").

### 4. Summarize Changes

`<change-summary>` forwarded → reuse verbatim, skip re-analysis. Otherwise derive from commits ahead of `<remote-base>`: review messages, group into 2-4 themes, summarize what and why (not how) → `<change-summary>`.

### 5. Resolve Ticket

`<ticket-mode>` already set → don't ask. Otherwise, `question` tool available → ask one question (header `Provide Ticket`, options `Automatically Create` / `Skip`, custom answers enabled); no tool → `<ticket-mode>` = `skip`, `<ticket-url>` = `SKIPPED`. Normalize: `Automatically Create` → `auto`; custom URL → `provided`; `Skip` → `skip`.

### 6. Load PR Template — BLOCKER

**Mandatory gate — do not compose `<pr-body>` until this completes. Most common failure mode.**

Probe all repo PR-template locations once. Keep the selected template verbatim in `<pr-template>`; set it to `none` only when the completed probe finds none. Step 7 fills it.

### 7. Generate Metadata + Prepare Ticket Reference

Generate `<pr-title>`, `<issue-summary>`, `<pr-body>`, and (when `auto`) `<ticket-title>`/`<ticket-body>` from `<resolved-base>`, `<ticket-mode>`, `<ticket-url>` (when `provided`), `<additional-context>`, `<pr-template>`.

`<pr-title>`: `<prefix>: <message>` — lowercase, under 50 chars. Prefixes: feat, fix, chore, docs, refactor, test, style; prefix = the dominant change type.

`<issue-summary>`: 1-2 sentences explaining the problem the PR solves. For `provided`, use the forwarded `<ticket-brief-result>` if it matches `<ticket-url>`, including a blocker as the final lookup result. Only without a matching result, get a ticket brief (see Ticket Brief) for that reference, scoped to the problem the PR addresses. No usable brief → note the gap outside the PR and infer from `<changes>` without fetching the ticket inline. For `auto` or `skip`, infer from `<changes>` directly. Include only details appropriate for the PR audience. Restate the problem even when linking `<ticket-url>`.

`<ticket-mode>` = `auto`: reuse `<change-summary>` themes from actual commits/diff. Title ≤70 chars reflecting the delivered outcome; description = what and why; checklists = 2-4 functional sections + final `Validation` section (reviewer-facing: "Verify that...", "Confirm that..."). Create via git host create issue (title, body, assignee `@me`); CLI fallback: pass `<ticket-body>` inline with shell quoting that preserves its contents. No attribution lines. Issue URL → `<ticket-url>`.

Otherwise: `provided` → use it; `skip` → `SKIPPED`.

**Compose `<pr-body>` from `<pr-template>`:** ≠ `none` → template verbatim as skeleton; fill placeholders/sections (`## Ticket`, `## Description`, `## Checklist`) from `<changes>` + `<ticket-url>` + `<issue-summary>`; preserve structure, headings, reviewer checkboxes — fill only, don't restructure; do NOT use the Step-9 hardcoded body. `<issue-summary>` must appear in the body: merge into the template's description section, else append a `## Issue` section — the sole permitted structure change. `none` → the Step-9 default structure.

### 8. Push Branch

Push with upstream via git host push or `git push -u origin <current-branch>` (`git-hosts.md`). Report `Push: yes/no` (`no` if "Everything up-to-date") → `<push-status>`, `<pushed-line>`.

### 9. Create PR

Use `<pr-title>` + `<pr-body>` (already reflects `<pr-template>`). Body when `<pr-template>` = `none`:
```markdown
## Ticket
<ticket-url>

## Issue
<issue-summary>

## Description
<short-description>

## Checklist
<checklist-items>
```
- `<pr-template>` ≠ `none` → the filled template from Step 7. No attribution lines. Preserve markdown newlines (single-line strings render `\n` literally).
- Create via git host create PR (title, body, base, head, assignee `@me`). CLI fallback: pass `<pr-body>` inline with shell quoting that preserves its contents.
- PR exists → git host view PR (pr_ref=`<current-branch>`, fields=`url,number`) or `gh pr view <current-branch> --json url,number`.
- URL → `<pr-url>`. Verify the rendered body (fields=`body` or `gh pr view <current-branch> --json body | jq -r '.body'`); re-edit if needed.
- Output the PR URL, created or existing.

### 10. Link PR in Ticket(s)

Skip when `<ticket-url>` = `SKIPPED` AND `<pr-body>` references no issues.

Reuse the creation workflow's authorization for verified, in-scope tickets. A stray or ambiguous reference is not consent to post; ask only if the target or intended link remains unresolved.

Collect every unique issue reference: `<ticket-url>` (unless `SKIPPED`) + issue URLs or `#N`/`OWNER/REPO#N` in `<pr-body>` (`#N` resolves against the current repo). For each verified, in-scope reference, post `PR: <pr-url>` via the issue tracker update-ticket operation (`issue-trackers.md`); CLI fallback `gh issue comment <issue-ref> --body "PR: <pr-url>"`. Report any ticket lacking a matching tracker operation — don't skip silently.

## Ticket Brief

Reuse current session evidence that covers the references, scoped answers, and decision status. Otherwise dispatch one isolated, read-only worker with the references, scope, `<project-root>`, these rules, and the target PR when known. The parent receives the brief only. If the worker or spec is unavailable, report the gap without fetching inline.

Worker: read `<project-root>/docs/issue-trackers.md`. Dedupe by tracker and ID; read title, body, checklists, and requested metadata. Read the latest comment page when scope includes behavior or constraints. Read older comments, attachments, and linked tickets only to close a scope gap or settle an unclear decision; this overrides general tracker traversal. For the target PR, return relevant decision references without fetching its feedback. Follow other PR links only for specific evidence the scope needs. Ignore unrelated bundled content; leave tracker state unchanged.

Return cited answers, decision status (confirmed, proposed, or superseded), blockers, and evidence gaps. Cite tickets, comment permalinks, or attachment names and locations; omit raw threads and attachments. Mark answers partial if unseen material could change them. Stop when the scope is supported.
