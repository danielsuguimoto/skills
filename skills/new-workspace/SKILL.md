---
name: new-workspace
description: "Use when the user prefixes a request or skill invocation with 'new-workspace' or asks to do the following work in a separate/isolated workspace — e.g. '/new-workspace /implement <card>' — or invokes it standalone to prepare an isolated workspace for subsequent work. Sets up an isolated workspace via the using-workspaces skill, then runs the requested skill or task inside it."
---

## Required `<project-root>/docs` reads

Read via shell **before** creating the workspace — may be gitignored and absent from the new workspace, so the copy in context stays the source of truth after the move. Missing → fall back per `using-workspaces`, note the gap; never invent contents.

- `<project-root>/docs/workspace-isolation.md`

# New Workspace

Companion skill: isolate first, then run whatever came next. Parse the invocation, set up, hand off.

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
| Workspace creation fails (sandbox/permissions) | Tell the user; ask whether to run `<downstream>` in place via the interactive question tool when one exists — don't silently proceed |
| `<downstream>` names an unknown skill | Set up, report, treat the text as a plain task |
