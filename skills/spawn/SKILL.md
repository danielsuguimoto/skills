---
name: spawn
description: >
  Spawn a subagent to run a raw prompt or invoke a target skill. The
  subagent's working context stays out of the parent; the parent gets the
  result. Use when a subagent dispatch is warranted (see the subagents skill
  for dispatch rationales); spawning a subagent to invoke a skill is usually
  an anti-pattern.
---

Spawn a subagent to run work off-thread. The subagent either executes a raw prompt directly or invokes a target skill. The harness infers which from the input — no explicit mode flag.

## Inputs

- `<input>` — a target skill identifier (as the harness addresses skills) optionally followed by arguments, OR a raw prompt for the subagent to execute directly.

## Procedure

1. Inspect `<input>`. If its leading token matches a registered skill name (via the harness's skill-listing mechanism), treat it as a skill invocation: `<skill-name>` = that token, `<arguments>` = the remainder. Otherwise treat the entire `<input>` as a raw prompt. Do NOT read the target skill's manifest from the parent.
2. Discover subagent profiles via the harness's native mechanism. Default: general-purpose profile, full tool access, foreground.
3. Spawn a subagent via the harness's native mechanism:
   - Skill invocation: prompt the subagent to invoke the target skill via the harness's skill-invocation mechanism with `<skill-name>` and `<arguments>`, then execute under that skill's directives. Front-load any context the target needs beyond `<arguments>`.
   - Raw prompt: prompt the subagent to execute `<input>` directly. Front-load all context the task needs; subagents are stateless.
4. Collect the subagent's return via the harness's native result-reading mechanism and output it.

## Output

The subagent's raw return.
