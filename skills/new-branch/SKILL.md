---
name: new-branch
description: "Use when starting new work that needs a branch — creates and switches to a categorized branch summarizing current uncommitted or ahead-of-base work. Invoke when the user says 'new branch', 'start a branch', or when beginning a new task."
---

# Branch From Work

Create and switch to a categorized branch summarizing current uncommitted work.

## MCP Operations

| Operation | Preferred MCP | CLI fallback |
|-----------|--------------|--------------|
| Load change state | `change_scan` | `git branch --show-current` + `git status --short` + `git diff --cached --stat` + `git diff --stat` + `git log origin/<branch>..<branch> --oneline` |
| Pull, create, publish | `branch_create` (with `branch_name`) | `git pull origin <branch>` + `git checkout -b <cat>/<slug>` + `git push -u origin <new-branch>` |

Follow AGENTS.md Git Conventions for all git/gh operations.

### 1. Interpret Arguments

- Wording influencing branch name → `<branch-context>`

### 2. Load Changes

If a pre-scan was forwarded, use `<current-branch>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, and `<branch-category>` directly. Skip to Step 3.

Otherwise:

Step 1: Load change state via the MCP table above. Store `<current-branch>` and `<uncommitted>`.

Step 2: If `<uncommitted>` is empty, check commits ahead using `origin/<current-branch>..<current-branch>`. Store `<ahead-commits>`.

Step 3: Analyze scope:
- Primary scope: `<uncommitted>`; fallback: `<ahead-commits>`.
- Group into logical themes. Summarize what and why, not how.
- Store as `<change-summary>`.

### 3. Check Blockers

- Nothing to branch from: `<uncommitted>` AND `<ahead-commits>` empty → STOP: "Nothing to branch from."
- Already on work branch: `<current-branch>` starts with `feature/`, `fix/`, `refactor/`, `docs/`, `test/`, `chore/`, `feat/`, `bugfix/`, `hotfix/`, `perf/`, `build/`, or `ci/` → STOP: "Branching skipped because the current branch already looks like a work branch."

### 4. Determine Category and Slug

- If `<branch-category>` was forwarded from pre-scan, reuse it; only generate the slug.
- Otherwise choose a category from `<change-summary>` and `<branch-context>`:
  - `feature` — new functionality
  - `fix` — bug fixes
  - `refactor` — restructuring without behavior change
  - `docs` — documentation
  - `test` — test-only
  - `chore` — tooling, dependencies, maintenance
  - `perf` — performance
  - `build` — build system
  - `ci` — CI/CD
- Store `<branch-category>`.
- Generate a concise kebab-case slug (2-4 words, descriptive) from `<change-summary>` and `<branch-context>`. Store as `<branch-slug>`.

### 5. Pull Latest from Remote

Pull `origin/<current-branch>` via MCP or CLI. If the pull fails (diverged history, no remote tracking), continue with the current state and note it in the output.

### 6. Create and Checkout Branch

Create `<branch-category>/<branch-slug>` via MCP `branch_create` or `git checkout -b`. If the branch exists, retry once with a numeric suffix (`-2`). Store as `<new-branch>`.

### 7. Publish Branch to Remote

Push with upstream via MCP `branch_create` or `git push -u origin <new-branch>`. `git push` only pushes committed changes; uncommitted changes stay local. Store `<published>` as `true` or `false`. Output the new branch name when created.

