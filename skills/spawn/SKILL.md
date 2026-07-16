---
name: spawn
description: >
  Invoke a target skill inside a subagent so its content loads only into the
  subagent's context, never the parent's.
---

Invoke a target skill inside a subagent. The skill body stays in the subagent's context; the parent gets the result.

## Inputs

- `<skill-name>` — target skill identifier (as the harness addresses skills).
- `<arguments>` — arguments forwarded verbatim.

## Procedure

1. Confirm `<skill-name>` exists via the harness's skill-listing mechanism. Missing → STOP, report it. Do NOT read the target's manifest from the parent.
2. Discover subagent profiles via the harness's native mechanism. Default: general-purpose profile, full tool access, foreground.
3. Spawn a subagent via the harness's native mechanism. Prompt it to invoke the target skill via the harness's skill-invocation mechanism with `<skill-name>` and `<arguments>`, then execute under that skill's directives. Front-load any context the target needs beyond `<arguments>`.
4. Collect the subagent's return via the harness's native result-reading mechanism and output it. The target skill's body never enters parent context.

## Output

The subagent's raw return.
