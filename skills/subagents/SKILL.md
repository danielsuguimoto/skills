---
name: subagents
description: "Subagent spawning gates. MANDATORY before any spawn, any large context-gathering scan (codebase, DB, docs, memories), or any work splittable into parallel independent tasks."
---

## Required `<project-root>/docs` reads

Shell-read before spawning any subagent; may be gitignored — missing → note gap, never invent: `<project-root>/docs/code-navigation.md`

Discover profiles through the tool's native mechanism — don't assume a fixed list or path. Verify availability before delegating; missing → fall back to inline.

## Dispatch Rationales

Spawn **only** when at least one holds — first two are strict; when in doubt, stay inline.

| Rationale | When to delegate |
|-----------|------------------|
| Parallel time win | **≥3 independent workstreams**, no shared writes. Below 3, coordination overhead eats the gain. |
| Context offloading | Project scanning whose raw output is **verifiably large** (state the estimated token/line count) AND the parent needs only a distilled brief. "Might bloat" is not a rationale. |
| Explicit user request | User asks to spawn/delegate/parallelize — honor even when no gate fires. |

## Pre-Spawn Checklist (fill inline before every dispatch)

1. **Rationale** — which gate fires? State it.
2. **Parallelism** — if parallel-time-win: list the ≥3 independent workstreams; confirm no shared writes.
3. **Offload size** — if context-offloading: state the raw output's estimated size; confirm only a distilled brief is needed.
4. **Profile** — name it; confirm it matches task shape (scout ≠ strong model, synthesis ≠ fast model).
5. **Foreground/background** — foreground default; background only if all background conditions hold (state them).

Can't fill all five → do the work inline.

## Do NOT spawn for

- Anything answerable from context already in this session.
- A single file edit, symbol lookup, or known path + line range.
- A bounded next step where the parent holds all inputs.
- Avoiding a thinking step.
- **Invoking a skill workflow** (commit, push, PR-create, ship-changes, branch creation). Skills run inline unless a dispatch rationale fires independently — spawning a subagent *to invoke a skill* is the most common anti-pattern.

## Foreground Preference

Background subagents have reduced capabilities: unapproved tools auto-denied, `exec` unavailable, no new permission prompts — they fail silently on approval-gated tasks. Use background only when ALL hold: ≥3 independent workstreams · needed tools pre-approved or read-only (grep, glob, read) · parent has productive work meanwhile. Otherwise foreground.

## Model Routing

Route by task shape — strong models cost latency and tokens; weak models fail complex reasoning.

| Task shape | Tier |
|---|---|
| Code exploration, scout brief, symbol trace | Fast / cheap |
| Doc fetch, library/SDK lookup | Mid |
| Implementation on disjoint files, PR-fix, synthesis | Strong |
| Memory extraction, to-specs planning | Mid–strong |
| Test suite execution | Fast |

**Defaults:** no `model` → inherit parent model; auto-routed → trust it, override only on clear mismatch; unsupported → inline or accept inheritance (never spawn a second subagent to work around it).

## Parallel Execution

**Don't parallelize** when: streams share file writes (serialize/merge) · one feeds the next (serialize/fan-in) · sync cost exceeds time saved · the work is a bounded single stream.

Patterns: **fan-out/fan-in** (N≥3 independent → wait all → optional synthesis subagent) · **pipeline** (A feeds B → serialize) · **scatter/gather** (N≥3, parent synthesizes inline when results are small).

## Dispatch Contract (every spawn)

- Front-load all context (paths, symbols, what's known, what to return) — subagents are stateless, no mid-run questions.
- Define the return shape explicitly; vague prompts produce vague returns.
- State write vs. read-only — mismatch is the most common delegation failure.
- Parent retains all git/gh and side-effecting operations unless explicitly authorized.

## Code Exploration

Read-only reconnaissance; the subagent returns a distilled brief — the parent never reads raw traversal. Specify depth and return shape (brief, file:line citations, diagram, named symbol set). Pick profile by intent: orientation vs. targeted question vs. architectural friction.

`BLOCKED`/`NO_*`/`UNAVAILABLE` → do the work inline; don't retry.

## Offloading Exception (hard gate, overrides stay-inline)

A task needing **>5 file reads spanning >2 modules** for exploration/tracing → dispatch a read-only scout BEFORE any inline `read`/`grep`/navigation query. The parent acts on the brief; raw contents never enter parent context.

| State | Scan shape | Action |
|---|---|---|
| Cold scan — parent doesn't hold the area | Multi-file / multi-module | Subagent scout |
| Warm read — parent holds the bulk | A few lines, one symbol, one file | Inline — dispatch overhead exceeds savings |

Inline exceptions: exact path + line range known · a single navigation symbol lookup/reference call answers it · target file <~50 lines and isolated.
