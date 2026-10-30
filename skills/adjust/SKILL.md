---
name: adjust
description: "Apply user-requested adjustments to existing code — fixes, improvements, refactors per notes. Interactive: grills the user to clarify the request, then parses, plans, and executes."
---

# Adjust

Apply the user's adjustment request to existing code — works on the code as it exists; uncommitted, clean, or no VCS are all irrelevant.

## Scope Boundary

Act on the request only: fix bugs, apply improvements, refactor per notes. Do NOT:

- Perform any version control operation — commit, push, branch switch, PR, merge.
- Reply to review threads (none exist).
- Suggest follow-up work, next steps, or shipping on completion.
- Widen scope unless the request explicitly asks.

The user ships separately.

## Quick Reference

- Input: `$ARGUMENTS` = user adjustment request (free-form prose, possibly multi-point), parsed directly.
- Workspace: stay in the current working directory.

## Grilling Discipline

Vague requests produce wrong changes — grill before parsing, planning, implementing.

- **One question at a time** — wait for the answer, never batch, via the interactive question tool when one exists.
- **Recommended answer** per question — one line, no justification.
- **Ground in evidence** — code can answer it → read first; ask only what evidence can't settle.
- **Cite file:line** when referencing existing code.
- **Stop when the target is met.** AFK → proceed with recommended answers, state assumptions.

### 1. Capture the Request

Read `$ARGUMENTS` → `<adjust-raw>`. Empty or only a question → STOP, ask for concrete adjustments.

### 2. Grill: Clarify the Request

One question at a time, rotate until each settles:

- **Intent** — desired outcome, restated in one line and confirmed.
- **Target** — file(s)/symbol(s): confirm named ones against code; locate candidates and ask otherwise.
- **Scope** — explicitly out of scope; existing behavior that must not change.
- **Priority** — multi-point → which matters most; recommend ordering.
- **Conflict check** — request contradicting an invariant/pattern/design decision → surface with the decisive snippet first.

Store as `<request-context>`. Intent still undefined after one round → **stop, list unknowns** — don't act speculatively.

### 3. Parse Into Actionable Items

Split `<adjust-raw>` + `<request-context>` into `<adjust-item>`s: `id` (`A1`, `A2`, …), `file`, `lines`, `category` (`critical`|`important`|`style`), `suggestion` (one-line imperative), `conflicts-with-design` (`yes` → defend current code, record reason, do NOT apply). `noise`/`question`/`conflicts-with-design: yes` → `<not-applying>` list.

### 4. Plan Implementation

Fast path: every actionable item is a specific suggestion with no conflicts → `<implementation-plan>` = `trivial`, skip to Step 5. Otherwise order `critical` → `important` → `style`, then file, then line; validate each (`invalid` if it breaks behavior or contradicts a contract → `<not-applying>`); map cross-item dependencies → `<dependencies>`.

### 5. Grill: Confirm the Plan

Before implementing:

- **The plan** — present ordered `<adjust-item>` + `<not-applying>` lists; proceed?
- **Scope check** — restate what will NOT change; any adjacent behavior to preserve?
- **Rollback** — how to undo on regression; recommend minimal, isolated changes.

Declines → Step 4, revise. Never implement an unconfirmed plan.

### 6. Implement

Critical first, existing patterns, minimal edits, no comments unless asked → `<changes-count>`.

### 7. Validate Changes

Run the project's lint and test commands → `<validation-results>`, `<validation-passing>` (`yes`/`no`). Confirm changes address the request points; missed intent → re-implement that item.

### 8. Present Results

Concise summary, then stop: one line per applied change · validation status · `<not-applying>` with one-line reasons · `<changes-count>` if tracked.
