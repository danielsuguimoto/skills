---
name: using-workspaces
description: "Use when starting work that needs an isolated workspace — feature branches, parallel tasks, or implementation plans. Detects existing isolation, then creates one per `<project-root>/docs/workspace-isolation.md`."
---

## Required `<project-root>/docs` reads

Shell-read before creating any workspace; may be gitignored — missing → follow Step 1c, note gap, never invent: `<project-root>/docs/workspace-isolation.md`

Ensure work happens in an isolated workspace — ordering: configured mechanism → harness capability → spec fallback. Never fight the harness.

## Step 0: Detect Existing Isolation

Check whether you're already isolated before creating anything:

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

`GIT_DIR != GIT_COMMON` → a linked workspace — unless it's a submodule. Verify:

```bash
# A path here means submodule, not a managed workspace — treat as a normal repo
git rev-parse --show-superproject-working-tree 2>/dev/null
```

VCS checks don't rule out tool-managed isolation: snapshot/copy mechanisms produce plain repos where `GIT_DIR == GIT_COMMON`. Also run the configured mechanism's *detect* operation from `workspace-isolation.md` (marker file, `ancestors`/`list` command, or harness API).

Already isolated → Step 2; report path and branch state. A detached HEAD is a valid externally-managed workspace — a branch is needed only at ship time (`new-branch`/`ship-changes`).

Not isolated and no declared user preference → ask consent first:

> "Would you like me to set up an isolated workspace? It protects your current checkout from changes."

Honor a declared preference without asking; declined → work in place, skip to Step 2.

## Step 1: Create Isolated Workspace

### 1a. Configured mechanism (preferred)

The mechanism in `workspace-isolation.md` is the project's choice — follow its detect/create/list/remove operations and semantics exactly; never guess commands.

### 1b. Harness-provided capability

Spec names no mechanism but the harness provides one (dedicated tool/command/flag) → use it. Harness capabilities own placement, branching, cleanup — bypassing them creates phantom state the harness can't see.

### 1c. Documented fallback

Only when neither applies — the fallback in the spec's Tool Details. Around whichever mechanism runs:

- **Directory**: explicit user-declared location wins; else the spec's documented location; else a project-local hidden directory. Project-local → verify ignored first (`git check-ignore -q <dir>`); not ignored → add to `.gitignore` and commit before creating.
- **Sandbox**: permission error on creation → tell the user the sandbox blocked it; work in place, run setup there.

Spec documents no fallback either → ask which mechanism to use; offer to write it into `workspace-isolation.md`.

## Step 2: Project Setup

Auto-detect and run setup inside the workspace from the manifests present. Copy/snapshot mechanisms may omit regenerable artifacts (`node_modules`, `target`, `dist`, caches) — dependency install is mandatory there even when the source checkout was fully set up.

Report `Workspace ready at <full-path>`.
