---
name: spawn
description: "Spawn a subagent to run a raw prompt or invoke a target skill — its context stays out of the parent; the parent gets the result. Use when a subagent dispatch is warranted (see subagents for rationales); spawning one to invoke a skill is usually an anti-pattern."
---

Spawn a subagent to run work off-thread — raw prompt or target-skill invocation; the harness infers which from the input.

## Inputs

- `<input>` — a target skill identifier (as the harness addresses skills) optionally followed by arguments, OR a raw prompt for the subagent to execute directly.

## Procedure

1. Inspect `<input>`. If its leading token matches a registered skill name (via the harness's skill-listing mechanism), treat it as a skill invocation: `<skill-name>` = that token, `<arguments>` = the remainder. Otherwise treat the entire `<input>` as a raw prompt. Do NOT read the target skill's manifest from the parent.
2. Discover subagent profiles via the harness's native mechanism; default: general-purpose, full tool access, foreground.
3. Spawn via the same mechanism — skill invocation: prompt it to invoke `<skill-name>` + `<arguments>` via the harness's skill-invocation mechanism and execute under that skill's directives, front-loading any needed context; raw prompt: execute `<input>` directly — front-load all context; subagents are stateless.
4. Return its raw output.
