---
name: grill-with-codebase
description: "Grill an unresolved code task, ticket, or idea into a buildable plan; also use on explicit grill requests."
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

Establish scope, success, exclusions, and preserved behavior first. Resolve relevant decisions about existing patterns and placement; data flow, persistence, scopes, and migrations; invariants and lifecycle hooks; edge cases and failures; callers, contracts, events, jobs, and downstream effects; and test impact. Consider concurrency, soft-delete, company scoping, partial failures, and rollbacks where applicable.

Record each decision. Every branch resolved → present the concrete plan: file-by-file changes, new artifacts, migrations, tests, open risks.
