---
name: debug
description: "Investigate and fix a reported bug, error, or unexpected behavior."
---

## Required `<project-root>/docs` reads

Shell-read `<project-root>/docs/code-navigation.md` before symbol lookups. Read `bug-trackers.md`, `memory-providers.md`, `observability-tools.md`, `database-tools.md`, or `doc-lookup.md` from the same directory only when the investigation needs that category. Specs may be gitignored — missing → note gap, never invent.

Troubleshoot and fix a user-reported issue. For read-only root-cause reporting use `triage`.

## Grilling Discipline

Use `grill-with-codebase` only for material decisions the request and evidence cannot resolve. Ask for the symptom or target first when it is too unclear to locate the code.

## Phase 1 — Context Gathering

Clarify an unclear symptom before exploring; otherwise proceed.

**1.1 Clarify the Symptom** — capture the symptom, expected and actual behavior, reproduction, and relevant environment from the request and available evidence. Quote errors verbatim; ask only for missing details needed to investigate → `<issue-context>`.

**1.2 Gather Evidence** — reproduce the exact symptom when feasible and trace the relevant code path. Inspect persisted state, logs, runtime, memory, or framework docs only when the evidence calls for them. Record failed reproduction attempts and findings with `file:line` citations → `<investigation-findings>`.

**Done when:** reproduction was attempted or its blocker recorded, the relevant path was traced, and the evidence and gaps are recorded.

## Phase 2 — Planning

Use sequential thinking when competing hypotheses need investigation; otherwise follow the evidence directly.

**2.1 Choose a Strategy** — use the evidence to choose the smallest fix and relevant verification. Ask only when a material decision remains unresolved → `<plan-decisions>`.

**2.2 Hypotheses** — when the cause is uncertain, rank plausible, falsifiable causes from `<investigation-findings>`. Do not invent alternatives to meet a count.

**2.3 Test Hypotheses** — one variable at a time; each probe maps to a prediction. Tag debug logs `[DEBUG-a4f2]` (cleanup = one grep). Targeted logs at boundaries — never "log everything and grep."

**2.4 Analyze Root Cause** — synthesize `<root-cause>`, `<affected-components>`, `<impact-scope>`, `<reproduction-conditions>`, `<potential-fixes>` (ranked).

**Done when:** evidence supports the cause and fix, material alternatives are resolved, and remaining gaps are named.

## Phase 3 — Execution

Minimal, surgical, verified.

**3.1 Resolve Decisions** — ask only if the fix needs a user decision or confirmation. Otherwise apply the requested fix.

**3.2 Implement** — failing test reproducing the bug first (when agreed); minimal change at the identified file/symbol/location; match existing style/naming/error-handling; no comments, speculative abstractions, or refactors beyond the fix; remove every `[DEBUG-xxxx]` log.

**3.3 Verify** — run checks appropriate to the fix. Reproduce the original scenario when feasible; report verification gaps. Confirm no instrumentation remains → `<validation-results>`, `<validation-passing>`.

**3.4 Present and Stop** — root cause (one line + citation) · fix applied (one line per change, file:line) · validation status · remaining gaps/risks. STOP — no commit, push, PR, or unsolicited next steps. User invokes `commit-and-push`/`ship-changes` to ship.

**Done when:** the fix and appropriate checks are complete, verification gaps are reported, instrumentation is removed, and results are presented.
