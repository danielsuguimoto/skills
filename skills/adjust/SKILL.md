---
name: adjust
description: "Apply requested fixes, improvements, or refactors to existing code; clarify unresolved decisions before implementation."
---

# Adjust

Works on the code as it exists — uncommitted, clean, or no VCS are all irrelevant.

## Scope Boundary

Act on the request only — no VCS operations, no review-thread replies, no follow-up suggestions, no scope widening unless asked. The user ships separately.

## Quick Reference

- Input: `$ARGUMENTS` = user adjustment request (free-form prose, possibly multi-point), parsed directly.

## Grilling Discipline

Use `grill-with-codebase` only for unresolved decisions that code and the request cannot answer. Proceed directly when scope and intent are clear.

### 1. Capture the Request

Read `$ARGUMENTS` → `<adjust-raw>`. Empty or only a question → STOP, ask for concrete adjustments.

### 2. Grill: Clarify the Request

Resolve these from the request and code. Ask only about material unknowns:

- **Intent** — desired outcome.
- **Target** — confirm named files or symbols against code; locate candidates otherwise.
- **Scope** — explicitly out of scope; existing behavior that must not change.
- **Priority** — multi-point → which matters most; recommend ordering.
- **Conflict check** — request contradicting an invariant/pattern/design decision → surface with the decisive snippet first.
- **Spec check** — a spec card reachable (ticket ref in `$ARGUMENTS`/conversation, or linked from the loaded ticket) → load it (shell-read `<project-root>/docs/issue-trackers.md`; take body, Implementation/Validation checklists, patches comment). A request point contradicting a spec item or a locked `Solution` decision → **divergence**: show the spec line vs. the request, ask whether the spec should change. Record each as `<spec-divergence>` (`confirmed`/`declined`); additions the spec simply doesn't cover aren't divergences.

Store as `<request-context>`. Intent still undefined after one round → **stop, list unknowns** — don't act speculatively.

### 3. Parse Into Actionable Items

Split `<adjust-raw>` + `<request-context>` into `<adjust-item>`s: `id` (`A1`, `A2`, …), `file`, `lines`, `category` (`critical`|`important`|`style`), `suggestion` (one-line imperative), `conflicts-with-design` (`yes` → defend current code, record reason, do NOT apply). An item tracing to a `declined` `<spec-divergence>` → `<not-applying>`; a `confirmed` one → flag `diverges-spec`, which exempts it from `conflicts-with-design` rejection — the user already approved overriding the spec'd design. `noise`/`question`/`conflicts-with-design: yes` (and not `diverges-spec`) → `<not-applying>` list.

### 4. Plan Implementation

Fast path: every actionable item is a specific suggestion with no conflicts → `<implementation-plan>` = `trivial`, skip to Step 5. Otherwise order `critical` → `important` → `style`, then file, then line; validate each (`invalid` if it breaks behavior or contradicts a contract → `<not-applying>`); map cross-item dependencies → `<dependencies>`.

### 5. Resolve Plan Decisions

Ask only when the plan requires a user decision or confirmation. Otherwise implement the requested changes without presenting a plan.

### 6. Implement

Critical first, existing patterns, minimal edits, no comments unless asked → `<changes-count>`. Each `diverges-spec` item also updates the spec card (sync per `issue-trackers.md`): rewrite the matching checklist items, patch blocks, and `Solution`/`Notes` lines so the spec reflects the applied change.

### 7. Validate Changes

Run checks appropriate to the changes → `<validation-results>`, `<validation-passing>` (`yes`/`no`). Confirm changes address the request points; missed intent → re-implement that item.

### 8. Present Results

Concise summary, then stop: one line per applied change · spec-card updates when applied · validation status · `<not-applying>` with one-line reasons · `<changes-count>` if tracked.
