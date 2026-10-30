---
name: pr-create
description: "Use when creating or opening a pull request for the current branch — standalone or as a ship-changes subskill."
---

## Required `<project-root>/docs` reads

Read via shell before creating the PR — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/git-hosts.md`
- `<project-root>/docs/issue-trackers.md`

## Operations

MCP tools take business-level inputs (title, body, base, head). Never use CLI flags. Skip `--body-file` temp-file steps when using MCP.

Operations: detect branch+default base, check uncommitted, fetch base+compute diff, push branch, create issue, create PR, view PR, check auth — all git host operations (`<project-root>/docs/git-hosts.md`). CLI fallback per `<project-root>/docs/git-hosts.md` for each operation. Ticket commenting uses the issue tracker update-ticket operation (`<project-root>/docs/issue-trackers.md`).

When run as a subskill, the invoking skill owns Steps 1-10. On "Nothing to ship.", proceed to Step 3 blockers.

### 1. Interpret Arguments

- Map inputs: base branch (e.g., `main`) → `<base>`; auto-create ticket → `<ticket-mode>` = `auto`; ticket URL/reference → `<ticket-url>` and `<ticket-mode>` = `provided`; skip ticket → `<ticket-mode>` = `skip`; guidance/focus areas → `<additional-context>`; no base → detect default branch in Step 2.

### 2. Load & Analyze Changes

- **Determine context.** Get the current branch via `git branch --show-current` (per `<project-root>/docs/git-hosts.md`). If `<base>` is not provided, detect the default via `git remote show origin | grep "HEAD branch" | awk '{print $NF}'` (per `<project-root>/docs/git-hosts.md`). Store `<current-branch>` and `<resolved-base>` (user `<base>` or detected default; strip `origin/` prefix). Use `<remote-base>` = `origin/<resolved-base>` for all comparisons; the local base is often stale.
- **Load changes.** Run `git status --short` and `git fetch origin <resolved-base>`. Then diff against `<remote-base>` using **three-dot syntax** (`...`) to exclude base-branch-only changes: ahead log, `git diff --stat`, `git diff`. Store as `<changes>`. Three-dot is mandatory.

### 3. Check Blockers

- Uncommitted changes: STOP (report + list files). On base branch: STOP (suggest `git checkout -b <feature-name>`). No changes: STOP (report "Nothing to include in a PR.").

### 4. Summarize Changes

If `<change-summary>` was forwarded, reuse it verbatim and skip re-analysis. Otherwise derive it from commits ahead of `<remote-base>`: review commit messages, group into 2-4 logical themes, and summarize what and why (not how). Store as `<change-summary>`.

### 5. Resolve Ticket

- If `<ticket-mode>` is already set, don't ask. Otherwise, if the `question` tool is available, ask one question (header `Provide Ticket`) with options `Automatically Create` / `Skip` and custom answers enabled. If no `question` tool, set `<ticket-mode>` = `skip` and `<ticket-url>` = `SKIPPED`.
- Normalize: `Automatically Create` → `auto`; custom URL → `provided`; `Skip` → `skip`.

### 6. Load PR Template — BLOCKER

**Mandatory gate. Do not compose `<pr-body>` (Step 7) until this completes. Most common failure mode.**

Check the repo for a GitHub PR template. Store the raw text as `<pr-template>` (verbatim: headings, placeholders, reviewer checkboxes). Do not fill yet — Step 7 fills it. No template found → `<pr-template>` = `none`. Re-probe before accepting `none`.

### 7. Generate Metadata + Prepare Ticket Reference

Generate `<pr-title>`, `<pr-body>`, `<ticket-title>` (when `<ticket-mode>` = `auto`), and `<ticket-body>` (when `auto`) from `<resolved-base>`, `<ticket-mode>`, `<ticket-url>` (when `provided`), `<additional-context>`, and `<pr-template>` (from Step 6).

When `<ticket-mode>` = `auto`: reuse `<change-summary>` themes based on actual commits/diff. Title (max 70 chars) reflecting the delivered outcome. Description: what and why. Checklists: 2-4 functional sections plus a final `Validation` section (reviewer-facing: "Verify that...", "Confirm that..."). Create via git host create issue (`<project-root>/docs/git-hosts.md`, title, body, assignee `@me`) or CLI fallback per `<project-root>/docs/git-hosts.md` with a temp file. No attribution lines. Store the issue URL as `<ticket-url>`.

Otherwise: `provided` → use provided value; `skip` → `SKIPPED`.

**Compose `<pr-body>` based on `<pr-template>`:**
- When `<pr-template>` ≠ `none`: use it verbatim as the body skeleton. Fill placeholders/sections (e.g., `## Ticket`, `## Description`, `## Checklist`) with content derived from `<changes>` and `<ticket-url>`. Preserve template structure, headings, and required reviewer checkboxes verbatim — only fill content, don't restructure. Do NOT use the hardcoded body in Step 9.
- When `<pr-template>` = `none`: fall back to the default body structure in Step 9.

### 8. Push Branch

Push with upstream via git host push (`<project-root>/docs/git-hosts.md`) or `git push -u origin <current-branch>` per `<project-root>/docs/git-hosts.md`. Report `Push: yes/no` (report `no` if "Everything up-to-date"). Store `<push-status>` and `<pushed-line>`.

### 9. Create PR

Use `<pr-title>` and `<pr-body>` from Step 7. `<pr-body>` already reflects `<pr-template>` (filled template when present, default structure otherwise).
- Body (when `<pr-template>` = `none`):
```markdown
## Ticket
<ticket-url>

## Description
<short-description>

## Checklist
<checklist-items>
```
- When `<pr-template>` ≠ `none`, use the filled template from Step 7 as the body. No attribution lines. Preserve markdown newlines (single-line strings render `\n` literally).
- Create via git host create PR (`<project-root>/docs/git-hosts.md`, title, body, base, head, assignee `@me`). CLI fallback per `<project-root>/docs/git-hosts.md`: write the body to a temp file, then run `gh pr create --title "<pr-title>" --body-file /tmp/pr-body.md --base "<resolved-base>" --head "<current-branch>" --assignee "@me"`.
- If the PR exists, use git host view PR (`<project-root>/docs/git-hosts.md`, pr_ref=`<current-branch>`, fields=`url,number`) or `gh pr view <current-branch> --json url,number` per `<project-root>/docs/git-hosts.md`.
- Store the URL as `<pr-url>`. Verify the rendered body via git host view PR (fields=`body`) or `gh pr view <current-branch> --json body | jq -r '.body'` per `<project-root>/docs/git-hosts.md`; re-edit if needed.
- Output the PR URL when created or if it already exists.

### 10. Link PR in Ticket(s)

Gate: skip when `<ticket-url>` = `SKIPPED` AND `<pr-body>` references no issues.

Collect every unique issue reference: `<ticket-url>` (when not `SKIPPED`) plus any other issue URLs or `#N`/`OWNER/REPO#N` references in `<pr-body>` (`#N` resolves against the current repo). For each, post the comment `PR: <pr-url>` via the issue tracker update-ticket operation (`<project-root>/docs/issue-trackers.md`). CLI fallback per `<project-root>/docs/issue-trackers.md`: `gh issue comment <issue-ref> --body "PR: <pr-url>"`. Report any ticket lacking a matching tracker operation instead of skipping silently.

