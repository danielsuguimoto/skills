---
name: new-workspace
description: "Use when a request or skill invocation is prefixed with 'new-workspace' (e.g. '/new-workspace /implement <card>') or should run in a separate/isolated workspace. Sets up isolation via the using-workspaces skill, then runs the requested work inside it."
---

## Required `<project-root>/docs` reads

Shell-read **before** creating the workspace; may be gitignored and absent from the new workspace — the in-context copy stays source of truth after the move. Missing → fall back per `using-workspaces`, note gap, never invent: `<project-root>/docs/workspace-isolation.md`

Companion skill: isolate first, then run whatever came next.

## 1. Parse the Invocation

Everything after `new-workspace` is downstream input:

- **Work request** — skill invocation (`/implement <card>`), named skill + args, or explicit task ("fix the logout bug") → `<downstream>`.
- **Context only** — bare URL, ticket link, file path, reference with no action → informs setup (e.g. naming); not a task.
- Empty or context-only → standalone mode (Edge Cases).

## 2. Set Up the Workspace

Invoke `using-workspaces` and follow it fully — consent is implied; skip its consent question.

## 3. Hand Off

All downstream work happens inside the workspace:

- Resolve `<project-root>` to the workspace root for code, tests, commands — except `<project-root>/docs` specs, which may exist only in the source checkout.
- Report the workspace path once, then run `<downstream>` as if invoked directly inside it — load and follow that skill in full, including its required reads. Run work only when the invocation named it; the workspace existing is the deliverable otherwise.

## Edge Cases

| Situation | Action |
|-----------|--------|
| Already inside an isolated workspace | Skip creation; run `<downstream>` in place and say so |
| No downstream request, or context-only input | Standalone mode: set up, report path, stop — subsequent instructions run inside the workspace |
| Workspace creation fails (sandbox/permissions) | Tell the user; ask whether to run `<downstream>` in place — don't silently proceed |
| `<downstream>` names an unknown skill | Set up, report, treat the text as a plain task |
