---
name: grill-with-codebase
description: "Use when a task, ticket, or vague intent touching a codebase needs interrogation into a buildable plan, or on 'grill' trigger phrases."
---

I have a task or intent, not a finished plan. Interview me until the answers form a concrete plan. Resolve dependencies between decisions one-by-one — order the chain as sequential thinking; each settled answer revises what stays open.

Rules:

- One question at a time — wait for the answer, via the interactive question tool when one exists.
- Recommended answer per question: one line, no justification.
- Read-only — the output is a plan, enacted only after I confirm it.

## Codebase is the ground truth

The task touches an existing codebase. When the target is identifiable, read the relevant code before asking questions. If the symptom or target is too unclear to locate it, ask the minimum needed first. Ground later questions and recommendations in current code, not generic best practice.

- A question answerable by exploring the codebase → explore instead of asking. Grill me only on decisions the code can't settle.
- A question or answer referencing existing code → cite `path:lineRange` so we both look at the same thing.
- I propose a shape contradicting an existing pattern, invariant, scope rule, or lifecycle → surface the conflict with the decisive snippet before moving on.
- Verify every claim about current behavior against code read this session; can't verify → say so and go read it.

## From task to plan

Establish scope and success state first, then rotate through:

- **Scope & success** — what "done" looks like; explicitly out of scope; behavior that must not change.
- **Fit with existing patterns** — follows surrounding conventions or silently introduces a new shape; where the new code lives.
- **Data flow & persistence** — where new data comes from, is stored, who reads it, scopes, migrations needed.
- **Invariants & lifecycle** — what must stay true; which subscriber/observer/lifecycle hooks the task breaks or bypasses.
- **Edge cases & failure modes** — null/empty inputs, concurrent writes, soft-delete, company scoping, partial failures, rollbacks.
- **Cross-module wiring** — who calls the changed code, who implements the contracts, what events/jobs fire, what breaks downstream.
- **Test impact** — which existing tests change; which new tests are demanded; what would regress untested.

Record each decision. Every branch resolved → present the concrete plan: file-by-file changes, new artifacts, migrations, tests, open risks.
