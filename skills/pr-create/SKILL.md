---
name: pr-create
description: "Use when creating or opening a pull request for the current branch — standalone or as a ship-changes subskill."
---

## Required `<project-root>/docs` reads

Shell-read before creating the PR; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`, `<project-root>/docs/issue-trackers.md`

## Operations

MCP tools take business-level inputs (title, body, base, head) — no CLI flags, no `--body-file` temp files. All git host ops per `git-hosts.md` (detect branch+default base, check uncommitted, fetch base+diff, push, create issue, create/view PR, auth). Ticket comments via issue tracker update-ticket (`issue-trackers.md`).

As a subskill, the invoking skill owns Steps 1-10; on "Nothing to ship.", proceed to Step 3 blockers.

### 1. Interpret Arguments

Base branch → `<base>`; auto-create ticket → `<ticket-mode>` = `auto`; ticket URL/reference → `<ticket-url>` + `<ticket-mode>` = `provided`; skip ticket → `skip`; guidance/focus areas → `<additional-context>`; no base → detect default in Step 2.

### 2. Load & Analyze Changes

- **Context.** `git branch --show-current` → `<current-branch>` (`git-hosts.md`). No `<base>` → detect default base per spec fallback. `<resolved-base>` = user `<base>` or detected default (strip `origin/`). Use `<remote-base>` = `origin/<resolved-base>` for all comparisons — the local base is often stale.
- **Changes.** `git status --short` + `git fetch origin <resolved-base>`; diff against `<remote-base>` with **three-dot syntax** (`...`) — mandatory, excludes base-branch-only changes: ahead log, `git diff --stat`, `git diff` → `<changes>`.

### 3. Check Blockers

Uncommitted changes → STOP (report + list files). On base branch → STOP (suggest `git checkout -b <feature-name>`). No changes → STOP ("Nothing to include in a PR.").

### 4. Summarize Changes

`<change-summary>` forwarded → reuse verbatim, skip re-analysis. Otherwise derive from commits ahead of `<remote-base>`: review messages, group into 2-4 themes, summarize what and why (not how) → `<change-summary>`.

### 5. Resolve Ticket

`<ticket-mode>` already set → don't ask. Otherwise, `question` tool available → ask one question (header `Provide Ticket`, options `Automatically Create` / `Skip`, custom answers enabled); no tool → `<ticket-mode>` = `skip`, `<ticket-url>` = `SKIPPED`. Normalize: `Automatically Create` → `auto`; custom URL → `provided`; `Skip` → `skip`.

### 6. Load PR Template — BLOCKER

**Mandatory gate — do not compose `<pr-body>` until this completes. Most common failure mode.**

Check the repo for a GitHub PR template → `<pr-template>` verbatim (headings, placeholders, reviewer checkboxes). Don't fill — Step 7 does. None found → `<pr-template>` = `none`; re-probe before accepting `none`.

### 7. Generate Metadata + Prepare Ticket Reference

Generate `<pr-title>`, `<pr-body>`, and (when `auto`) `<ticket-title>`/`<ticket-body>` from `<resolved-base>`, `<ticket-mode>`, `<ticket-url>` (when `provided`), `<additional-context>`, `<pr-template>`.

`<ticket-mode>` = `auto`: reuse `<change-summary>` themes from actual commits/diff. Title ≤70 chars reflecting the delivered outcome; description = what and why; checklists = 2-4 functional sections + final `Validation` section (reviewer-facing: "Verify that...", "Confirm that..."). Create via git host create issue (title, body, assignee `@me`) or CLI fallback with temp file. No attribution lines. Issue URL → `<ticket-url>`.

Otherwise: `provided` → use it; `skip` → `SKIPPED`.

**Compose `<pr-body>` from `<pr-template>`:** ≠ `none` → template verbatim as skeleton; fill placeholders/sections (`## Ticket`, `## Description`, `## Checklist`) from `<changes>` + `<ticket-url>`; preserve structure, headings, reviewer checkboxes — fill only, don't restructure; do NOT use the Step-9 hardcoded body. `none` → the Step-9 default structure.

### 8. Push Branch

Push with upstream via git host push or `git push -u origin <current-branch>` (`git-hosts.md`). Report `Push: yes/no` (`no` if "Everything up-to-date") → `<push-status>`, `<pushed-line>`.

### 9. Create PR

Use `<pr-title>` + `<pr-body>` (already reflects `<pr-template>`). Body when `<pr-template>` = `none`:
```markdown
## Ticket
<ticket-url>

## Description
<short-description>

## Checklist
<checklist-items>
```
- `<pr-template>` ≠ `none` → the filled template from Step 7. No attribution lines. Preserve markdown newlines (single-line strings render `\n` literally).
- Create via git host create PR (title, body, base, head, assignee `@me`). CLI fallback: body to temp file, `gh pr create --title "<pr-title>" --body-file /tmp/pr-body.md --base "<resolved-base>" --head "<current-branch>" --assignee "@me"`.
- PR exists → git host view PR (pr_ref=`<current-branch>`, fields=`url,number`) or `gh pr view <current-branch> --json url,number`.
- URL → `<pr-url>`. Verify the rendered body (fields=`body` or `gh pr view <current-branch> --json body | jq -r '.body'`); re-edit if needed.
- Output the PR URL, created or existing.

### 10. Link PR in Ticket(s)

Skip when `<ticket-url>` = `SKIPPED` AND `<pr-body>` references no issues.

Collect every unique issue reference: `<ticket-url>` (unless `SKIPPED`) + issue URLs or `#N`/`OWNER/REPO#N` in `<pr-body>` (`#N` resolves against the current repo). For each, post `PR: <pr-url>` via the issue tracker update-ticket operation (`issue-trackers.md`); CLI fallback `gh issue comment <issue-ref> --body "PR: <pr-url>"`. Report any ticket lacking a matching tracker operation — don't skip silently.
