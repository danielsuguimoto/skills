---
name: sequential-thinking
description: "Use sequential reasoning for complex debugging, multi-step planning, ambiguous scope, or competing hypotheses."
---

For non-trivial work, reason in a chain of numbered thoughts. Each advances, revises, or branches the investigation; do not narrate actions.

## Mechanism

Use a todo list for reasoning-only work, inline reasoning for short chains, or a configured thinking tool (`<project-root>/docs/mcp-servers.md`) when persistent branches justify it. One thought per todo; encode revisions and branches in the item when nesting is unsupported. If the todo list tracks execution, keep reasoning inline or in a thinking tool. Tool availability never gates the method.

## The Loop

1. **Frame** the problem and estimate the chain length; extend as needed, never pad.
2. **Advance** one job per thought. When evidence contradicts earlier reasoning, mark the revision and the thought it revises.
3. **Branch** unresolved alternatives; pursue the strongest while retaining the others until falsified.
4. **Verify** hypotheses against predictions and evidence; mark each confirmed or falsified.
5. **Close** on a verified conclusion or a decision for the user. The last thought is the answer.
