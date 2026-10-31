---
name: using-workspaces
description: "Use when starting work that needs an isolated workspace — feature branches, parallel tasks, or running an implementation plan. Detects existing isolation, then creates a workspace via the project's configured mechanism (see `<project-root>/docs/workspace-isolation.md`), a harness-provided capability, or the fallback documented in the spec."
---

## Required `<project-root>/docs` reads

Read via shell before creating any workspace — may be gitignored. Missing → follow Step 1c, note the gap; never invent contents.

- `<project-root>/docs/workspace-isolation.md`

# Using Workspaces

Ensure work happens in an isolated workspace — one concept, several backends. Never assume the mechanism: prefer the project's configured one, then a harness-provided capability, then the documented fallback. Never fight the harness. Ask via the interactive question tool when one exists.

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

VCS checks don't rule out tool-managed isolation: snapshot/copy mechanisms produce plain repos where `GIT_DIR == GIT_COMMON`. Also run the configured mechanism's *detect* operation from `<project-root>/docs/workspace-isolation.md` (marker file, `ancestors`/`list` command, or harness API).

Already isolated → Step 2; report path and branch state. A detached HEAD is a valid externally-managed workspace — a branch is needed only at ship time (`new-branch`/`ship-changes`).

Not isolated and no declared user preference → ask consent first:

> "Would you like me to set up an isolated workspace? It protects your current checkout from changes."

Honor a declared preference without asking; declined → work in place, skip to Step 2.

## Step 1: Create Isolated Workspace

Try in order; use the first that applies.

### 1a. Configured mechanism (preferred)

The mechanism in `<project-root>/docs/workspace-isolation.md` is the project's choice — follow its detect/create/list/remove operations and semantics exactly; never guess commands.

### 1b. Harness-provided capability

Spec names no mechanism but the harness provides one (dedicated tool/command/flag) → use it. Harness capabilities own placement, branching, cleanup — bypassing them creates phantom state the harness can't see.

### 1c. Documented fallback

Only when neither applies — the fallback in the spec's Tool Details. Around whichever mechanism runs:

- **Directory**: explicit user-declared location wins; else the spec's documented location; else a project-local hidden directory. Project-local → verify ignored first (`git check-ignore -q <dir>`); not ignored → add to `.gitignore` and commit before creating.
- **Sandbox**: permission error on creation → tell the user the sandbox blocked it; work in place, run setup there.

Spec documents no fallback either → ask which mechanism to use; offer to write it into `<project-root>/docs/workspace-isolation.md`.

## Step 2: Project Setup

Auto-detect and run setup inside the workspace from the manifests present. Copy/snapshot mechanisms may omit regenerable artifacts (`node_modules`, `target`, `dist`, caches) — dependency install is mandatory there even when the source checkout was fully set up.

Report `Workspace ready at <full-path>`.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "I'm obviously not in a workspace — no need to check" | Run Step 0 including the detect op. Harness isolation, snapshot copies, and submodules all fool eyeballing. |
| "Creating it directly is quicker than reading the spec" | The configured mechanism owns placement, hooks, cleanup. Bypassing it creates phantom state outside its registry — the #1 mistake. |
| "The workspace directory is surely ignored already" | Run `git check-ignore`. An unignored workspace directory commits the whole tree into the repo. |
| "A detached HEAD means something is broken" | Snapshot mechanisms intentionally copy with detached HEAD. Commit on it; create the branch when shipping. |
| "A quick test run proves the workspace is good" | Creation and setup are the deliverable; downstream work owns verification. |
