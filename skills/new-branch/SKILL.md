---
name: new-branch
description: "Use for 'new branch' requests or new work that needs a branch — creates and switches to a categorized branch from a description, ticket/issue link, or current uncommitted/ahead-of-base work, before or after implementation begins."
---

## Required `<project-root>/docs` reads

Shell-read before creating or switching to the branch; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`, `<project-root>/docs/issue-trackers.md` (only when a ticket/issue link argument is provided)

# Branch From Argument or Work

Create and switch to a categorized branch. All git/gh operations follow `git-hosts.md`.

| Operation | Tool | CLI fallback |
|-----------|------|--------------|
| Load change state | git host change scan | change-scan fallback per spec (ahead: `git log origin/<branch>..<branch> --oneline`) |
| Pull, create, publish | git host branch create (`branch_name`) | `git pull origin <branch>` + `git checkout -b <cat>/<slug>` + `git push -u origin <new-branch>` |

### 1. Interpret Arguments

Classify the argument (if any) → `<arg-mode>` (`description`|`ticket`|`none`) + `<branch-context>`:

- **Description** — free-form text, not a URL/issue link → use directly as `<branch-context>`.
- **Ticket/issue link** — URL or identifier referencing a ticket/issue → extract content via `issue-trackers.md` (title, summary, labels/type) → `<branch-context>`; extraction fails → fall back to a slug-relevant fragment (issue number, path slug), note the gap.
- **None** — no argument; `<branch-context>` empty; name derived from git changes (Step 2).

### 2. Load Changes

Always runs — git changes are validated every invocation.

Pre-scan forwarded → use `<current-branch>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<branch-category>` directly; skip to Step 3.

Otherwise: (1) load change state per the table → `<current-branch>`, `<uncommitted>`; (2) `<uncommitted>` empty → check ahead via `origin/<current-branch>..<current-branch>` → `<ahead-commits>`; (3) analyze scope — primary `<uncommitted>`, fallback `<ahead-commits>` — group into logical themes, summarize what and why (not how) → `<change-summary>`.

### 3. Check Blockers

- Nothing to branch from: `<arg-mode>` = `none` AND `<uncommitted>` AND `<ahead-commits>` all empty → STOP: "Nothing to branch from."
- Already on a work branch: `<current-branch>` has a `(feature|fix|refactor|docs|test|chore|feat|bugfix|hotfix|perf|build|ci)/` prefix → STOP: "Branching skipped because the current branch already looks like a work branch."

Note: `<arg-mode>` = `description`/`ticket` → missing git changes do NOT block; the branch is created from the argument and the empty change state is recorded as validation output.

### 4. Determine Category and Slug

`<branch-category>` forwarded from pre-scan → reuse; only generate the slug. Otherwise infer:

- `ticket` → from ticket type/labels first; fall back to `<change-summary>` + `<branch-context>`.
- `description` → from `<branch-context>`; fall back to `<change-summary>`.
- `none` → from `<change-summary>` + `<branch-context>`.

Categories: `feature` (new functionality) · `fix` (bug fixes) · `refactor` (restructure, no behavior change) · `docs` · `test` · `chore` (tooling/deps/maintenance) · `perf` · `build` · `ci`.

Slug: concise kebab-case, 2-4 descriptive words → `<branch-slug>`. Source: `description`/`ticket` → `<branch-context>` (consult `<change-summary>` only to disambiguate); `none` → `<change-summary>` + `<branch-context>`.

### 5. Pull Latest from Remote

`git pull origin <current-branch>` via git host or CLI. Pull fails (diverged, no tracking) → continue with current state, note it in output.

### 6. Create and Checkout Branch

`<branch-category>/<branch-slug>` via git host branch create or `git checkout -b` per the table. Branch exists → retry once with numeric suffix (`-2`) → `<new-branch>`.

### 7. Publish Branch to Remote

Push with upstream via git host or `git push -u origin <new-branch>`. Push only ships committed changes; uncommitted stay local → `<published>` `true`/`false`. Output the new branch name.
