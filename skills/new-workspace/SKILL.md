---
name: new-workspace
description: "Use when a request or skill invocation is prefixed with 'new-workspace' (e.g. '/new-workspace /implement <card>', '/new-workspace /pr-fix <pr>') or should run in a separate/isolated workspace — including work based on an existing PR."
---

## Required `<project-root>/docs` reads

Shell-read **before** creating the workspace; may be gitignored and absent from the new workspace — the in-context copy stays source of truth after the move. Missing → fall back per `using-workspaces`, note gap, never invent: `<project-root>/docs/workspace-isolation.md`, `<project-root>/docs/issue-trackers.md` (only when the invocation references a ticket/card), `<project-root>/docs/git-hosts.md` (only when it references a PR)

Companion skill: isolate first, then run whatever came next.

## 1. Parse the Invocation

Everything after `new-workspace` is downstream input:

- **Work request** — skill invocation (`/implement <card>`), named skill + args, or explicit task ("fix the logout bug") → `<downstream>`.
- **Context only** — bare URL, ticket link, file path, reference with no action → informs setup (naming — or the workspace base when it's a PR); not a task.
- Empty or context-only → standalone mode (Edge Cases).

Then derive `<workspace-name>` — a concise kebab-case slug naming the work:

- Input references a ticket/card (URL or ID) → load it via `issue-trackers.md` and slug from `<ticket-title>` (an ID alone is not a friendly name); keep the payload in context for `<downstream>` to reuse instead of loading it again. Load fails → fall back to the ID, note the gap.
- Input references an existing PR (URL or number) → `<pr-ref>`; several PRs → `<pr-ref>` is the one `<downstream>` targets, the rest stay context. Load `<pr-ref>` via `git-hosts.md` (`View PR`), slug from `<pr-title>`, and keep the payload in context for `<downstream>` to reuse instead of loading it again. Load fails → fall back to the PR number, note the gap.
- Free-text task or description → slug from that text.

## 2. Set Up the Workspace

Resolve the base ref first — the workspace must start from the tip of the remote's default branch (`master`/`main`), never the current checkout, which may be stale or a feature branch:

```bash
git fetch origin
git remote set-head origin --auto 2>/dev/null
BASE=$(git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null || echo origin/main)
```

Then invoke `using-workspaces` and follow it fully — consent is implied; skip its consent question. Pass `origin/<default>` as the creation base and `<workspace-name>` as the name wherever the mechanism accepts them; a mechanism that fixes the base to the current checkout → flag it to the user, don't silently base on it.

A workspace is ready when its `HEAD` points at the resolved base commit — compare via git host operations (`git-hosts.md`). A detached `HEAD` satisfies this; never require a `master`/`main` checkout or drop a mechanism-installed worktree link to get one. A task branch is created only when the work ships (`new-branch`/`ship-changes`).

Invocation references a `<pr-ref>` → point the workspace at the PR branch between creation and project setup:

- A mechanism sharing local branch refs (linked worktree) can't attach the PR head branch while another worktree holds it — check that against the `View PR` payload and the source checkout first; held → prefer a mechanism with independent refs, or report the blocker and stop before readiness or handoff.
- After creation, run the `Checkout PR` git host operation (`git-hosts.md`) inside the workspace — it attaches the existing PR head branch, not a new task branch. The readiness target becomes `HEAD` on that branch at the PR head commit; verify it before `using-workspaces`' project-setup step so dependencies install from the PR.

## 3. Hand Off

All downstream work happens inside the workspace:

- Resolve `<project-root>` to the workspace root for code, tests, commands — except `<project-root>/docs` specs, which may exist only in the source checkout.
- Report the workspace path once, then run `<downstream>` as if invoked directly inside it — load and follow that skill in full, including its required reads. Run work only when the invocation named it; the workspace existing is the deliverable otherwise.

## Edge Cases

| Situation | Action |
|-----------|--------|
| Already inside an isolated workspace | Skip creation; run `<downstream>` in place and say so — a `<pr-ref>` still triggers `Checkout PR`, but only on a clean worktree and index; dirty → leave the checkout alone, ask the user to preserve the changes |
| PR fails to load (missing, auth) | Tell the user; ask whether to proceed on the default branch or stop — a PR-targeting `<downstream>` will retry the PR on its own |
| No downstream request, or context-only input | Standalone mode: set up, report path, stop — subsequent instructions run inside the workspace |
| Workspace creation fails (sandbox/permissions) | Tell the user; ask whether to run `<downstream>` in place — don't silently proceed |
| `<downstream>` names an unknown skill | Set up, report, treat the text as a plain task |
