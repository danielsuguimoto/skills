---
name: debug
description: "Use when a user reports a bug, error, or unexpected behavior and wants it troubleshot and fixed. Interactive: grills the user to clarify the problem, then investigates, plans, and executes the fix."
---

## Required `<project-root>/docs` reads

Shell-read before investigating the bug; may be gitignored — missing → note gap, never invent: `<project-root>/docs/bug-trackers.md`, `<project-root>/docs/code-navigation.md`, `<project-root>/docs/database-tools.md`, `<project-root>/docs/doc-lookup.md`, `<project-root>/docs/memory-providers.md`, `<project-root>/docs/observability-tools.md`, `<project-root>/docs/terminal-wrappers.md`, `<project-root>/docs/terminal-tools.md`

Troubleshoot and fix a user-reported issue. Three phases, each with a grilling section. This skill **executes fixes**; for read-only root-cause reporting use `triage`.

## Tool Agnosticism

Reference `<project-root>/docs` specs, never hardcode tool names: code symbols → `code-navigation.md`, DB → `database-tools.md`, framework docs → `doc-lookup.md`, error logs → `bug-trackers.md`, observability → `observability-tools.md`, terminal → `terminal-wrappers.md`, memory → `memory-providers.md` (all under `<project-root>/docs/`).

## Grilling Discipline

Grill per `grill-with-codebase` before exploring, hypothesizing, or fixing. AFK → proceed with recommended answers, state assumptions.

## Phase 1 — Context Gathering

Don't explore until the symptom is clear.

**1.1 Grill: Clarify the Symptom** — rotate until each settles: Symptom (quote error verbatim) · Expected vs actual · Reproduction (exact steps; no repro → last known-working + what changed) · Environment (version, branch, deployment, data state, feature flags) · Scope (who affected, when started) · Frequency (always, intermittent, load/time-dependent) → `<issue-context>`. Still undefined after one round → stop, list unknowns.

**1.2 Gather Evidence** — collect, don't guess: Reproduce against the exact symptom (nearby failure isn't the bug; repro fails → say so, list attempts). Trace code path user action → failure (`code-navigation.md`). Inspect DB when state persists (`database-tools.md`) — live data is ground truth, don't trust seeders/migrations/factories. Pull error logs (`bug-trackers.md`), read full trace. Inspect app runtime (`observability-tools.md`). Check memory for prior pitfalls (`memory-providers.md`). Fetch framework docs when behavior hinges on version/API (`doc-lookup.md`) → `<investigation-findings>` with file:line citations.

**Done when:** symptom reproduced or attempted · code path traced · DB verified when relevant · traces read in full · context + findings stored.

## Phase 2 — Planning

Strategy before code — the first plausible idea is usually not the cause.

**2.1 Grill: Confirm the Strategy** — Hypotheses (present ranked list, confirm/reorder) · Fix approach (minimal patch, refactor, or workaround — recommend one) · Risk tolerance (what must not change — recommend safest boundary) · Test strategy (recommend a failing test reproducing the bug) → `<plan-decisions>`.

**2.2 Ranked Hypotheses** — 3–5 from `<investigation-findings>`, each falsifiable: "If `<X>` is the cause, then `<changing Y>` makes the bug disappear." No prediction → vibe, discard. Show the ranked list before testing; don't block if AFK.

**2.3 Test Hypotheses** — one variable at a time; each probe maps to a prediction. Tag debug logs `[DEBUG-a4f2]` (cleanup = one grep). Targeted logs at boundaries — never "log everything and grep."

**2.4 Analyze Root Cause** — synthesize `<root-cause>`, `<affected-components>`, `<impact-scope>`, `<reproduction-conditions>`, `<potential-fixes>` (ranked).

**Done when:** 3–5 falsifiable hypotheses · winner tested, alternatives falsified · cause/components/scope/conditions stated with evidence · fixes ranked · plan confirmed (or assumptions if AFK) · gaps named.

## Phase 3 — Execution

Minimal, surgical, verified.

**3.1 Grill: Confirm Before Applying** — the fix (chosen `<potential-fixes>` entry + exact files/symbols, cited — proceed?) · scope check (what will NOT change) · rollback (how to undo; minimal and isolated). Declines → Phase 2. Never apply an unconfirmed fix.

**3.2 Implement** — failing test reproducing the bug first (when agreed); minimal change at the identified file/symbol/location; match existing style/naming/error-handling; no comments, speculative abstractions, or refactors beyond the fix; remove every `[DEBUG-xxxx]` log.

**3.3 Verify** — lint + tests: reproducing test passes, no regression. Reproduce the original scenario manually when feasible — symptom gone. Confirm no instrumentation remains → `<validation-results>`, `<validation-passing>`.

**3.4 Present and Stop** — root cause (one line + citation) · fix applied (one line per change, file:line) · validation status · remaining gaps/risks. STOP — no commit, push, PR, or unsolicited next steps. User invokes `commit-and-push`/`ship-changes` to ship.

**Done when:** confirmed fix applied at cited file/symbol · reproducing test passes, no regressions · symptom verified gone · instrumentation removed · results presented.


