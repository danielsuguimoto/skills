---
name: debug
description: "Use when a user reports a bug, error, or unexpected behavior and wants it troubleshot and fixed. Interactive: grills the user to clarify the problem, then investigates, plans, and executes the fix."
---

Troubleshoot and fix a user-reported issue. Three phases, each with a grilling section. This skill **executes fixes**; for read-only root-cause reporting use `triage`.

## Tool Agnosticism

Reference `/docs` specs in the project root, never hardcode tool names:

| Need | Spec |
|------|------|
| Code symbols, references, tracing | `/docs/code-navigation.md` |
| Database schema, queries, sample rows | `/docs/database-tools.md` |
| Framework/library/SDK behavior | `/docs/doc-lookup.md` |
| Error logs, stack traces, bug records | `/docs/bug-trackers.md` |
| Terminal commands | `/docs/terminal-wrappers.md` |
| Memory of prior pitfalls | `/docs/memory-providers.md` |

Missing spec → fall back to native tools, note the gap.

## Grilling Discipline

Vague reports waste investigation. Grill before exploring, before hypothesizing, before fixing.

- **One question at a time.** Wait for the answer. Never batch.
- **Recommended answer** with each question — one line, no justification.
- **Ground in evidence.** If codebase, logs, or DB can answer it, investigate first; ask only what evidence cannot settle.
- **Cite file:line** when a question references existing code.
- **Stop when the target is met.** Don't interrogate past clarity.
- **AFK** → proceed with recommended answers, state assumptions.

## Phase 1 — Context Gathering

Don't explore the codebase until the symptom is clear.

### 1.1 Grill: Clarify the Symptom

One question at a time, rotate until each settles:

- **Symptom**: what happens? Quote the error verbatim, don't paraphrase.
- **Expected vs actual**: should-be vs is.
- **Reproduction**: exact trigger steps. No repro → ask last known-working state + what changed.
- **Environment**: version, branch, deployment, data state, feature flags.
- **Scope**: who's affected — one user, one company, all? When did it start?
- **Frequency**: always, intermittent, load-dependent, time-dependent?

Store as `<issue-context>`: `<symptom>`, `<expected>`, `<actual>`, `<reproduction-steps>`, `<environment>`, `<impact-scope>`, `<frequency>`.

Symptom still undefined after one round → **stop, list unknowns.** Don't investigate speculatively.

### 1.2 Gather Evidence

`<issue-context>` settled. Collect, don't guess:

- **Reproduce** against the user's exact symptom. A nearby failure isn't the bug. Repro fails → say so, list what was tried.
- **Trace the code path** user action → failure (see `/docs/code-navigation.md`). Find where actual diverges from expected.
- **Inspect the DB** when state persists (see `/docs/database-tools.md`). Live data is ground truth — schema, counts, sample rows, actual values. Don't trust seeders, migrations, factories.
- **Pull error logs and traces** (see `/docs/bug-trackers.md`). Read the full trace, not just the top frame.
- **Check memory** for prior pitfalls on this module/pattern (see `/docs/memory-providers.md`).
- **Fetch framework docs** when behavior hinges on a version/API contract (see `/docs/doc-lookup.md`).

Store as `<investigation-findings>` with file:line citations.

### 1.3 Done when ALL hold

- [ ] Symptom reproduced or repro attempted with explicit results
- [ ] Code path traced entry → failure
- [ ] DB state verified (when relevant)
- [ ] Error traces read in full
- [ ] `<issue-context>` + `<investigation-findings>` stored

## Phase 2 — Planning

Strategy before code. Single-hypothesis thinking anchors on the first plausible idea and misses the cause.

### 2.1 Grill: Confirm the Strategy

One question at a time:

- **Hypotheses**: present the ranked list (below), ask user to confirm or reorder.
- **Fix approach**: minimal patch, broader refactor, or workaround? Recommend one.
- **Risk tolerance**: what existing behavior must not change? Recommend the safest boundary.
- **Test strategy**: new test, existing test, manual repro? Recommend a failing test that reproduces the bug.

Store as `<plan-decisions>`.

### 2.2 Ranked Hypotheses

Generate 3–5 from `<investigation-findings>`. Each falsifiable — state the prediction: "If `<X>` is the cause, then `<changing Y>` makes the bug disappear." No falsifiable prediction → vibe, discard.

Show the ranked list before testing. Don't block if AFK.

### 2.3 Test Hypotheses

Change one variable at a time. Each probe maps to a specific prediction. Tag debug logs with a unique prefix (e.g., `[DEBUG-a4f2]`) — cleanup is one grep. Never "log everything and grep"; targeted logs at boundaries that distinguish hypotheses.

### 2.4 Analyze Root Cause

Synthesize evidence:
- `<root-cause>` — fundamental reason
- `<affected-components>` — parts involved
- `<impact-scope>` — how widespread
- `<reproduction-conditions>` — triggering conditions
- `<potential-fixes>` — ranked approaches

### 2.5 Done when ALL hold

- [ ] 3–5 ranked hypotheses, each falsifiable
- [ ] Winning hypothesis tested, prediction held; alternatives falsified
- [ ] `<root-cause>`, `<affected-components>`, `<impact-scope>`, `<reproduction-conditions>` stated with evidence
- [ ] `<potential-fixes>` ranked
- [ ] `<plan-decisions>` confirmed (or assumptions stated if AFK)
- [ ] Gaps named if root cause can't be fully determined

## Phase 3 — Execution

Minimal, surgical, verified.

### 3.1 Grill: Confirm Before Applying

- **The fix**: state the chosen `<potential-fixes>` entry + exact files/symbols to change, with citations. Proceed?
- **Scope check**: state what will NOT change. Any adjacent behavior to preserve?
- **Rollback**: how to undo if it regresses. Recommend keeping the change minimal and isolated.

User declines → return to Phase 2, revise `<potential-fixes>`. Don't apply an unconfirmed fix.

### 3.2 Implement

Drive off `<plan-decisions>` and the confirmed fix:
- Write a failing test reproducing the bug first (when a test strategy was agreed).
- Apply the minimal change at the identified file/symbol/location.
- Match existing style, naming, error-handling.
- No comments unless asked. No speculative abstractions. No refactors beyond the fix.
- Remove every `[DEBUG-xxxx]` log from Phase 2.

### 3.3 Verify

- Run lint (see project config or `/docs/`).
- Run tests. Reproducing test passes; no existing test regresses.
- Reproduce the original scenario manually (when feasible) — symptom gone.
- Confirm no debug instrumentation remains.

Store `<validation-results>`, `<validation-passing>` (`yes`/`no`).

### 3.4 Present and Stop

- Root cause (one line) + evidence citation.
- Fix applied (one line per change, file:line).
- Validation status.
- Remaining gaps or follow-up risks.

STOP. No commit, no push, no PR, no unsolicited next steps. User invokes `commit-and-push` or `ship-changes` to ship.

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "I'll just fix it directly" | Fixing without root cause guarantees recurrence. |
| "The bug is obvious" | Obvious bugs are usually symptoms, not causes. |
| "Debugging takes too long" | Systematic debugging beats trial-and-error. |
| "I don't need to ask the user" | Vague report sent to the wrong target wastes the whole investigation. |

## Red Flags

**Never:**
- Propose or apply a fix before root cause is identified.
- Conclude root cause without a reproduction confirmed against the user's exact symptom.
- Generate a single hypothesis — anchor on the first plausible idea, miss the cause.
- State a hypothesis with no falsifiable prediction — vibes aren't hypotheses.
- Skip a grilling section. Vague input produces wrong fixes.
- Batch grilling questions. One at a time, then wait.
- Apply a fix the user has not confirmed in Phase 3.
- Commit, push, or open a PR — stops at presentation.
- Leave debug instrumentation in the codebase after the fix.
- Guess at root cause without evidence.

**Always:**
- Grill before exploring, before hypothesizing, before fixing.
- Stop and say so explicitly when you cannot reproduce. List what was tried, then ask.
- Generate 3–5 ranked hypotheses before concluding.
- Show the ranked list before testing (don't block if AFK).
- Tag debug logs with a unique `[DEBUG-xxxx]` prefix.
- Write a failing test reproducing the bug before the fix (when a test strategy was agreed).
- Run lint + tests after the fix; confirm no regression.
- Cite file:line in every finding.
- Ask "what would have prevented this?" and flag architectural gaps.
- Present results and stop — no unsolicited next steps.
