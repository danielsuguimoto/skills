---
name: using-todos
description: "Use the harness-native todo list for execution spanning three or more steps, multiple files, distinct phases, or parallel workstreams — except a single mechanical action."
---

Use the harness-native todo list as the execution ledger: the user-visible record of what remains, what is active, and what finished.

## Create

Create the list only when execution spans three or more steps, multiple files, distinct phases, or parallel workstreams. Skip a single mechanical action, even when it touches multiple files. No harness-native todo mechanism → use the workflow's own plan artifact as the ledger; without one, keep a brief outcome sequence inline.

## Items

- One item = one outcome the session must produce or verify; split compounds.
- Write each item as a short imperative naming the outcome and target.
- Order by dependency; keep one `in_progress` item per active execution stream — exactly one for serial work.
- Keep rich workflow metadata (file, symbol, patch, evidence, scenario, dependency) in the workflow's own artifact. When the native schema has only content and status, project one concise item per executable outcome instead of copying the artifact.
- When execution and reasoning both need tracking, reserve the native list for execution outcomes; keep the thought chain in its reasoning medium.
- Keep coverage maps, test plans, patch inventories, and ticket checklists in their workflows.

## Maintain

- Mark an item `in_progress` immediately before starting it; mark it `completed` immediately after its focused verification passes.
- A failed or blocked item stays `in_progress` while active or returns to `pending` when paused; report the blocker rather than marking unfinished work `completed`.
- Revise on drift: add discovered outcomes, split oversized items, reword drifted items, and reorder dependencies.
- Remove an item only when the user changes scope or evidence shows the outcome is already satisfied or no longer applicable; never silently drop requested work.
- When replacing the list, preserve completed items and current order unless drift requires revision.

## Parallel fan-out

- Use one item per independent workstream. Multiple items may be `in_progress` only while their streams run concurrently; the parent owns every status transition.
- Label each stream in `content` (`W1`, `W2`, …) when the native schema lacks ownership metadata.
- For blocking dispatch, mark each stream `in_progress` immediately before launching it; for nonblocking dispatch, mark it `in_progress` after launch is acknowledged. Mark it `completed` only after the parent validates its result; launch errors and failed or invalid results follow the Maintain rules.

## Close

Before the final response, reconcile every item: completed items have verification; unfinished requested work appears in the final report with its blocker or next action.
