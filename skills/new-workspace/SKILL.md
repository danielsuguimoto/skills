---
name: new-workspace
description: "Run a task or skill in an isolated workspace, including work on an existing PR; use on new-workspace requests."
---

## Required `<project-root>/docs` reads

Shell-read **before** creating the workspace; may be gitignored and absent from the new workspace — the in-context copy stays source of truth after the move. Missing → fall back per `using-workspaces`, note gap, never invent: `<project-root>/docs/workspace-isolation.md`, `<project-root>/docs/git-hosts.md`, `<project-root>/docs/issue-trackers.md` (only when the resolved target is a ticket/card)

Companion skill: isolate first, then run whatever came next.

## 1. Parse the Invocation

Everything after `new-workspace` is downstream input:

- **Work request** — skill invocation (`/implement <card>`), named skill + args, or explicit task ("fix the logout bug") → `<downstream>`.
- **Context only** — bare URL, ticket link, file path, or "workspace for that ticket" with no requested work → informs setup (naming — or the workspace base when it's a PR); not a task.
- Empty or context-only → standalone mode (Edge Cases).

When the invocation or `<downstream>` targets a PR or ticket without an explicit URL or ID, reuse the unambiguous relevant reference from the current session:

- **Ticket** — use the ticket URL or ID, such as the spec ticket returned by `to-specs`, for `/new-workspace /implement` or a standalone workspace for that ticket. Name the workspace from the ticket and start from the default branch; a ticket alone does not target a PR.
- **PR** — use the PR number or URL, such as the PR returned by `ship-changes`, as `<pr-ref>` for workspace setup and PR checkout.

Pass the resolved reference to `<downstream>` when present. Multiple possible targets → ask. No matching session target → follow `<downstream>`'s missing-input rules before setup, or ask for the reference when standalone; a required PR target must be resolved before creating the workspace. Do not attach a session reference to an unrelated or untargeted standalone workspace.

Then derive `<workspace-name>` — a concise kebab-case slug naming the work:

- Resolved target is a ticket/card (URL or ID) → load it via `issue-trackers.md` and slug from `<ticket-title>` (an ID alone is not a friendly name); keep the reference and payload in context for `<downstream>` to reuse instead of loading it again. Load fails → fall back to the ID, note the gap.
- Resolved target is an existing PR (URL or number) → `<pr-ref>`; several PRs → `<pr-ref>` is the one `<downstream>` targets, the rest stay context. Load `<pr-ref>` via `git-hosts.md` (`View PR`), slug from `<pr-title>`, and keep the reference and payload in context for `<downstream>` to reuse instead of loading it again. Load fails → fall back to the PR number, note the gap.
- Free-text task or description → slug from that text.

## 2. Set Up the Workspace

Resolve the base ref first — the workspace must start from the tip of the remote's default branch (`master`/`main`), never the current checkout, which may be stale or a feature branch:

```bash
git fetch origin
git remote set-head origin --auto 2>/dev/null
BASE=$(git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null || echo origin/main)
```

Then invoke `using-workspaces` with `<setup-mode>` = `create-only`, `<workspace-name>`, and the resolved base — consent is implied; skip its consent question. Require verified isolation before continuing; `using-workspaces` owns the base check.

Resolved target is a `<pr-ref>` → point the workspace at the PR branch between verified creation and project setup:

- A mechanism sharing local branch refs (linked worktree) can't attach the PR head branch while another worktree holds it — check that against the `View PR` payload and the source checkout first; held → report the blocker and stop before readiness or handoff. The configured mechanism still owns the choice (`using-workspaces`) — don't silently switch to an independent-refs one.
- After verified creation, run the `Checkout PR` git host operation (`git-hosts.md`) inside the workspace — it attaches the existing PR head branch, not a new task branch. The readiness target becomes `HEAD` on that branch at the PR head commit; verify it before `using-workspaces` Step 2 so dependencies install from the PR. A failed targeted checkout → report the blocker and stop; do not fall back to the base workspace without an explicit prior user choice.
- An untrusted `<pr-ref>` (fork, outside author) → project setup can run PR-authored install hooks as the local user; flag that and confirm with the user before `using-workspaces` Step 2 runs.

Run `using-workspaces` Step 2 (project setup) inside the verified workspace — for a `<pr-ref>` target only after the PR checkout is verified and any untrusted-PR confirmation is given — then hand off per Step 3.

## 3. Hand Off

All downstream work happens inside the workspace:

- Resolve `<project-root>` to the workspace root for code, tests, commands — except `<project-root>/docs` specs, which may exist only in the source checkout.
- Report the workspace path once, then run `<downstream>` as if invoked directly inside it — load and follow that skill in full, including its required reads. Run work only when the invocation named it; the workspace existing is the deliverable otherwise.

## Edge Cases

| Situation | Action |
|-----------|--------|
| Already inside an isolated workspace | Reuse the verified workspace without comparing its HEAD to the default base. If targeting a PR, require a clean worktree and index before the common PR checkout and trust gate. Run `<downstream>` in place only after any required PR checkout succeeds |
| PR fails to load (missing, auth) | Tell the user; ask whether to proceed on the default branch or stop — a PR-targeting `<downstream>` will retry the PR on its own |
| No downstream request, or context-only input | Standalone mode: set up, report path, stop — subsequent instructions run inside the workspace |
| Workspace creation fails (sandbox/permissions) | Tell the user and stop; run `<downstream>` in place only on an explicit user choice — no default fallback |
| `<downstream>` names an unknown skill | Set up, report, treat the text as a plain task |
