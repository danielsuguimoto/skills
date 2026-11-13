---
name: sequential-thinking
description: "Sequential thinking method for non-trivial tasks — decompose into a thought chain, revise on new evidence, branch competing hypotheses or approaches, verify before concluding. Invoke on complex debugging, multi-step planning, unclear-scope analysis, or any task where one-shot reasoning risks a wrong answer."
---

Externalize reasoning as a chain of numbered thoughts instead of reasoning in one pass. Each thought is a unit of work — a decomposition, an evidence read, a decision — not a narration of actions taken.

## Mechanism

The method needs no dedicated tool — pick whatever carries the chain:

- **Task/todo list** — one item per thought; updates mark revisions, nested items mark branches. Persists the chain across steps and context boundaries.
- **Inline reasoning** — run the loop mentally for short chains where tool overhead exceeds gain.
- **Thinking tool** — use a configured tool (`<project-root>/docs/mcp-servers.md`) when persistent branches or revisions justify its call overhead.

The loop binds either way — tool availability never gates it.

## The Loop

1. **Frame** — state the problem; estimate the thoughts needed. The estimate is a guess — extend when the problem proves deeper, never pad to reach it.
2. **Advance** — one thought, one job: decompose a subproblem, weigh evidence, compare options, conclude a step. A thought doing two jobs splits.
3. **Revise** — later evidence contradicting an earlier thought → mark it a revision of that thought. Silent restatement buries what changed; the chain keeps the correction visible.
4. **Branch** — competing explanations or approaches the evidence can't yet settle → fork each from the divergence point; pursue the strongest, keep the rest alive until falsified.
5. **Verify** — a hypothesis earns conclusion only through its prediction: derive what must hold, test it, mark confirmed or falsified.
6. **Close** — stop on a verified conclusion or a decision that belongs to the user. The last thought is the answer — no summary pass.

## Discipline

- Every thought advances, revises, or branches — one restating context or narrating a tool call is noise.
- Course correction is the point: a long chain on a hard problem with no revision or branch rationalizes the first idea rather than reasoning.
- Non-trivial only — a mechanical step or a question context already answers never opens a chain.
