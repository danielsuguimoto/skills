---
name: pr-create
description: "Creates a pull request for the current branch."
---

## Required `/docs` reads

Read these project-root spec files before acting (use shell `cat`/`ls` — they may be in `.gitignore`, invisible to built-in search). Missing file → fall back to native tools, note the gap; never invent contents.

- `/docs/git-hosts.md`

## Operations

MCP tools take business-level inputs (title, body, base, head). Never use CLI flags. Skip `--body-file` temp-file steps when using MCP.

Operations: detect branch+default base, check uncommitted, fetch base+compute diff, push branch, create issue, create PR, view PR, check auth — all git host operations (see `/docs/git-hosts.md` in the project root). CLI fallback per `/docs/git-hosts.md` for each operation.

Follow `/docs/git-hosts.md` in the project root for all git/gh operations.

The parent owns Steps 1-10. On `NOTHING_TO_SHIP`, proceed to Step 3 blockers.

### 1. Interpret Arguments

- Map inputs: base branch (e.g., `main`) → `<base>`; auto-create ticket → `<ticket-mode>` = `auto`; ticket URL/reference → `<ticket-url>` and `<ticket-mode>` = `provided`; skip ticket → `<ticket-mode>` = `skip`; guidance/focus areas → `<additional-context>`; no base → detect default branch in Step 1.

### 2. Load & Analyze Changes

Step 1: Determine context. Get the current branch via `git branch --show-current` (per `/docs/git-hosts.md` in the project root). If `<base>` is not provided, detect the default via `git remote show origin | grep "HEAD branch" | awk '{print $NF}'` (per `/docs/git-hosts.md` in the project root).
Store `<current-branch>` and `<resolved-base>` (user `<base>` or detected default; strip `origin/` prefix). Use `<remote-base>` = `origin/<resolved-base>` for all comparisons; the local base is often stale.

Step 2: Load changes. Run `git status --short` and `git fetch origin <resolved-base>`. Then diff against `<remote-base>` using **three-dot syntax** (`...`) to exclude base-branch-only changes: ahead log, `git diff --stat`, `git diff`. Store as `<changes>`. Three-dot is mandatory.

### 3. Check Blockers

- Uncommitted changes: STOP (report + list files). On base branch: STOP (suggest `git checkout -b <feature-name>`). No changes: STOP (report "Nothing to include in a PR.").

### 4. Summarize Changes

If `<change-summary>` was forwarded, reuse it verbatim and skip re-analysis. Otherwise derive it from commits ahead of `<remote-base>`: review commit messages, group into 2-4 logical themes, and summarize what and why (not how). Don't describe base-branch-only work. Store as `<change-summary>`.

### 5. Resolve Ticket

- If `<ticket-mode>` is already set, don't ask. Otherwise, if the `question` tool is available, ask one question (header `Provide Ticket`) with options `Automatically Create` / `Skip` and custom answers enabled. If no `question` tool, set `<ticket-mode>` = `skip` and `<ticket-url>` = `SKIPPED`.
- Normalize: `Automatically Create` → `auto`; custom URL → `provided`; `Skip` → `skip`.

### 6. Generate Metadata + Prepare Ticket Reference

Generate `<pr-title>`, `<pr-body>`, `<ticket-title>` (when `<ticket-mode>` = `auto`), and `<ticket-body>` (when `auto`) from `<resolved-base>`, `<ticket-mode>`, `<ticket-url>` (when `provided`), and `<additional-context>`.

When `<ticket-mode>` = `auto`: reuse `<change-summary>` themes based on actual commits/diff. Title (max 70 chars) reflecting the delivered outcome. Description: what and why. Checklists: 2-4 functional sections plus a final `Validation` section (reviewer-facing: "Verify that...", "Confirm that..."). Create via git host create issue (see `/docs/git-hosts.md` in the project root, title, body, assignee `@me`) or CLI fallback per `/docs/git-hosts.md` with a temp file. No attribution lines. Store the issue URL as `<ticket-url>`.

Otherwise: `provided` → use provided value; `skip` → `SKIPPED`.

### 7. Push Branch

Push with upstream via git host push (see `/docs/git-hosts.md` in the project root) or `git push -u origin <current-branch>` per `/docs/git-hosts.md`. Report `Push: yes/no` (report `no` if "Everything up-to-date"). Store `<push-status>` and `<pushed-line>`.

### 8. Load PR Template

Before composing the body, check the repo for a GitHub PR template. Globs: `.github/PULL_REQUEST_TEMPLATE*.md`, `.github/pull_request_template*.md`, `docs/PULL_REQUEST_TEMPLATE*.md`, `PULL_REQUEST_TEMPLATE*.md`. Detect it with `find_file_by_name` or git host file listing (see `/docs/git-hosts.md` in the project root) / `git ls-files` per `/docs/git-hosts.md`. If multiple match, prefer one matching the `<current-branch>` prefix (e.g., `PULL_REQUEST_TEMPLATE_feature.md` for `feature/*`); otherwise use the default template.

If a template is found, store it as `<pr-template>` and use it verbatim as the body skeleton. Fill placeholders/sections (e.g., `## Ticket`, `## Description`, `## Checklist`) with content derived from `<changes>` and `<ticket-url>`. Preserve template structure, headings, and required reviewer checkboxes verbatim — only fill content, don't restructure. Skip the hardcoded body in §9 Step 1; use the filled template instead.

If no template is found, set `<pr-template>` = `none`; fall back to the default body structure in §9.

### 9. Create PR

Use `<pr-title>` and `<pr-body>` from Step 6. When `<pr-template>` ≠ `none`, override `<pr-body>` with the filled template from §8; `<pr-title>` still applies. Title (max 70 chars) → `<pr-title>`. Description: brief intent/scope. Checklist: 2-4 functional sections plus `Validation`.
- Body (when `<pr-template>` = `none`):
```markdown
## Ticket
<ticket-url>

## Description
<short-description>

## Checklist
<checklist-items>
```
When `<pr-template>` ≠ `none`, use the filled template from §8 as the body. No attribution lines. Preserve markdown newlines (single-line strings render `\n` literally). Prefer git host create PR (see `/docs/git-hosts.md` in the project root, title, body, base, head, assignee `@me`). CLI fallback per `/docs/git-hosts.md`: write the body to a temp file, then run `gh pr create --title "<pr-title>" --body-file /tmp/pr-body.md --base "<resolved-base>" --head "<current-branch>" --assignee "@me"`. If the PR exists, use git host view PR (see `/docs/git-hosts.md` in the project root, pr_ref=`<current-branch>`, fields=`url,number`) or `gh pr view <current-branch> --json url,number` per `/docs/git-hosts.md`. Store the URL as `<pr-url>`. Verify the rendered body via git host view PR (fields=`body`) or `gh pr view <current-branch> --json body | jq -r '.body'` per `/docs/git-hosts.md`; re-edit if needed. Output the PR URL when created or if it already exists.

