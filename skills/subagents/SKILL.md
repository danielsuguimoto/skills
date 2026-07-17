---
name: subagents
description: >
  Subagent spawning gates. MANDATORY before any spawn, any large
  context-gathering scan (codebase, DB, docs, memories), or any work
  splittable into parallel independent tasks.
---

## Required `/docs` reads

Read these project-root spec files before acting (use shell `cat`/`ls` — they may be in `.gitignore`, invisible to built-in search). Missing file → fall back to native tools, note the gap; never invent contents.

- `/docs/code-navigation.md`

Discover profiles through the tool's native mechanism. Don't assume a fixed list or path.

## Dispatch Rationales

Spawn a subagent **only** when at least one holds. First two are strict — when in doubt, stay inline. Third always wins.

| Rationale | When to delegate |
|-----------|------------------|
| Parallel time win | **≥3 independent workstreams**, no shared writes. Below 3, coordination overhead eats the gain. |
| Context offloading | Task demands **project scanning** (multi-file exploration, doc fetches, memory bodies, Q&A transcripts) whose raw output would pollute parent context with material the parent never needs verbatim. |
| Explicit user request | User asks to spawn / delegate / parallelize. Honor even when neither gate fires. |

**Default to inline.** Subagents cost dispatch overhead, a stateless round-trip, and a synthesis step. Pay that only for a clear parallel win (≥3 streams) or to keep parent context clean.

## Foreground Preference

**Default to foreground.** Background subagents have reduced capabilities: unapproved tools auto-denied, `exec` unavailable, no new permission prompts. They often fail silently on tasks requiring tool approval.

Use background **only** when all hold:
- ≥3 truly independent workstreams benefit from parallelism
- Needed tools already pre-approved or read-only (grep, glob, read)
- Parent has productive work while waiting

Otherwise spawn foreground. Parent pauses but the subagent gets full tool access and can prompt for permissions.

Verify profile availability before delegating. If missing, fall back to inline.

## Model Routing

Route by task shape. Strong models cost latency and tokens; weak models fail complex reasoning.

| Task shape | Tier | Rationale |
|---|---|---|
| Code exploration, scout brief, symbol trace | Fast / cheap | Bounded read, distilled return |
| Doc fetch, library/SDK lookup | Mid | Broad retrieval, moderate synthesis |
| Implementation on disjoint files, PR-fix, synthesis | Strong | Multi-step reasoning, code generation |
| Memory extraction, to-specs planning | Mid–strong | Judgment + structured output |
| Test suite execution | Fast | Mechanical run + report |

Mechanism is tool-specific: profile frontmatter `model` field, agent config, enterprise "subagent router" setting, or unsupported (inherits parent model). Discover via the tool's native profile/config system.

**Defaults:** no `model` set → inherit parent session model; tool auto-routes → trust the router, override only on clear task-shape mismatch; unsupported → inline or accept inheritance, never spawn a second subagent to work around it.

**Never** pin a strong model on a scout. **Never** pin a fast model on a synthesis step.

## Parallel Execution

**Don't parallelize** when: workstreams share file writes (serialize/merge), one stream feeds the next (serialize/fan-in), sync cost exceeds time saved, or the work is a bounded single stream.

Patterns: **Fan-out / fan-in** (N≥3 independent → wait all → optional synthesis subagent), **Pipeline** (A's output feeds B → serialize), **Scatter / gather** (N≥3 subagents, parent synthesizes inline when results are small).

## Dispatch Contract (applies to every spawn)

- Front-load all context (file paths, function names, what's known, what to return). Subagents are stateless — no mid-run clarifying questions.
- Define return shape explicitly. Vague prompts produce vague returns.
- State write vs. read-only. Mismatch is the most common delegation failure.
- Parent retains all git/gh and side-effecting operations unless explicitly authorized.

## Code Exploration

Read-only reconnaissance. Subagent returns a distilled brief; parent never reads raw traversal. Specify depth and return shape (brief, file:line citations, diagram, named symbol set). Pick profile by intent: orientation vs. targeted question vs. architectural friction.

Fallback on `BLOCKED`: if a subagent returns `BLOCKED` / `NO_*` / `UNAVAILABLE`, do that work inline. Don't retry.

## Mandatory Offloading Threshold

Hard gate (overrides "default to inline"): if a task needs **more than 3 file reads** OR spans **more than 1 module** for exploration/tracing, dispatch a read-only scout BEFORE any inline `read`/`grep`/code navigation query (see `/docs/code-navigation.md` in the project root). The parent acts on the brief; raw file contents never enter parent context.

Exploration tokens are the largest avoidable parent-context cost. Once read, they stay in context for the whole session. A scout's distilled brief costs a fraction and is discardable.

**Cold scan vs. warm read (decides inline vs. subagent):**

| State | Parent context | Scan shape | Action |
|---|---|---|---|
| Cold scan | Doesn't hold the target area yet | Multi-file / multi-module exploration | Subagent scout — raw output would bloat parent |
| Warm read | Already holds the bulk of context for the task | Small partial scan (a few lines, one symbol, one file) to confirm or double-check | Inline — offloading costs more in dispatch overhead than it saves |

Subagents exist to keep cold-scan raw output out of parent context. A warm read adds negligible tokens because the parent already carries the surrounding context — dispatch overhead exceeds the savings.

Other exceptions (inline allowed): exact file path and line range already known; a single code navigation symbol lookup / reference trace call (see `/docs/code-navigation.md` in the project root) answers it; target file under ~50 lines and fully isolated.

## Failure Modes

- **Premature completion** — parent skips the dispatch gates to "just do it inline". Check the Dispatch Rationales first. Only inline if no rationale fires.
- **Vague return shape** — subagent dumps raw content. State the return shape in the prompt before dispatch.
- **Read-only / write mismatch** — subagent edits when it should read, or vice versa. State read-only vs. write in every dispatch prompt.
- **Retry loop** — parent re-dispatches a subagent that returned `BLOCKED`. Do the work inline. Don't retry.

**Never:** delegate without stating read-only vs. write; retry a `BLOCKED` subagent; let a scout dump raw file contents; parallelize workstreams that share file writes.

**Always:** front-load all context; check dispatch gates before inline work; parent retains all git/gh and side-effecting operations; delegate unbounded-cost work (docs, memory dumps, Q&A transcripts).
