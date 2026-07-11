---
name: grill-with-codebase
description: Grill the user about a task or intent, using the codebase as ground truth, until the resolved decisions form a concrete plan. Use when the user brings a task, ticket, or vague intent that touches an existing codebase and wants to be interrogated to shape it into a buildable plan, or uses any 'grill' trigger phrases alongside a codebase context.
---

I have a task or intent, not a finished plan. Interview me one question at a time until the answers form a concrete plan. Resolve dependencies between decisions one-by-one. Give a recommended answer with each question.

Ask one question at a time. Wait for feedback before continuing. Don't ask multiple questions at once.

Don't enact the plan until I confirm the plan is concrete.

## Codebase is the ground truth

The task touches an existing codebase. Before the first question, read the affected code: files, modules, contracts, callers, data flow, tests, and nearby invariants. Ground every question and recommended answer in what the code actually does today, not generic best practice.

- If a question can be answered by exploring the codebase, explore the codebase instead of asking. Grill me only on the decisions the code can't settle on its own.
- When a question or recommended answer references existing code, cite the file path and line range so we both look at the same thing.
- When I propose a shape that contradicts an existing pattern, invariant, scope rule, or lifecycle in the code, surface the conflict with the decisive snippet before moving on.
- Verify every claim about current behavior against code you actually read this session. If you can't verify it, say so and go read it before continuing.

## From task to plan

Start by establishing the task's scope and success state, then rotate through these angles:

- **Scope & success**: what does "done" look like; what is explicitly out of scope; what existing behavior must not change?
- **Fit with existing patterns**: does the task follow the surrounding conventions, or does it silently introduce a new shape? Where does the new code live?
- **Data flow & persistence**: where does new data come from, where is it stored, who reads it, what scopes apply, what migrations are needed?
- **Invariants & lifecycle**: what must stay true; which existing subscriber/observer/lifecycle hooks does the task break or bypass?
- **Edge cases & failure modes**: null/empty inputs, concurrent writes, soft-delete, company scoping, partial failures, rollbacks.
- **Cross-module wiring**: who calls the changed code, who implements the contracts, what events/jobs fire, what breaks downstream.
- **Test impact**: what existing tests change; what new tests the task demands; what behavior would regress untested.

Record each decision. When every branch resolves, present the concrete plan: file-by-file changes, new artifacts, migrations, tests, and open risks. Ask for confirmation before enactment.

## Delegation First (MANDATORY before inline grilling)

Before the inline grill loop, check the dispatch gate. Delegate ambiguous or under-specified tasks to a foreground subagent. The subagent runs the one-question loop off-context, spawns further subagents for codebase-answerable questions, and returns only resolved decisions — never the Q&A transcript. Run inline only if the subagent is unavailable or returns `BLOCKED`.

- One question at a time. Don't dump 5 questions in one message. Ask, wait, then next.
- Recommended answer: one line. Don't justify it.
- This skill is read-only. Don't edit, refactor, or fix anything while grilling. The output is a plan, not a change.
