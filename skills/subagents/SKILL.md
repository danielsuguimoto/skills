---
name: subagents
description: "Subagent spawning gates. MANDATORY before any spawn, any large context-gathering scan (codebase, DB, docs, memories), or any work splittable into parallel independent tasks."
---

A skill's `Ticket Brief` procedure delegates only the read-only ticket inspection. That procedure's isolation and blocker rules replace the sequential, size, and inline-fallback gates below — the parent never probes raw ticket content to qualify. Keep the calling workflow in the parent.

## Sequential Gate

Keep sequential work in the parent. If B needs A's result, run A then B inline. This gate applies to one blocking handoff as well as a chain of subagents; waiting for a child is not parallel work.

Only an explicit user request to delegate that work overrides this gate. Context offloading, model cost, role specialization, and foreground execution do not. Apply this gate before the dispatch rationales or profile discovery.

## Dispatch Rationales

After the sequential gate passes, spawn **only** when at least one holds — first two are strict; when in doubt, stay inline.

| Rationale | When to delegate |
|-----------|------------------|
| Parallel time win | **≥3 independent workstreams**, no shared writes. Below 3, coordination overhead eats the gain. |
| Context offloading | Exceptional read-only research that passes the context-offloading gate below. |
| Explicit user request | User asks to spawn/delegate/parallelize that work. A request to complete a task or use a skill is not delegation permission. |

### Context-Offloading Gate

Start inline with path listings, targeted searches, and bounded reads. A folder with 10 files is ordinary inline work; file count, repository size, unfamiliarity, and multiple modules do not justify delegation.

Offload only when those probes establish **all** of the following:

- The question requires inspecting **more than 10,000 tokens** of still-unread, task-relevant material. Name the sources and give a size estimate grounded in observed output; count relevant sections, not whole folders or unrelated file contents.
- Narrower searches or reads cannot answer the question. State why.
- The parent needs only a cited brief and will not reread the raw material.

Unknown size or uncertain necessity → stay inline. The size floor is necessary, not permission by itself; the sequential gate still applies.

## Pre-Spawn Checklist (fill inline before every dispatch)

1. **Rationale** — which gate fires? State it.
2. **Parallelism** — if parallel-time-win: list the ≥3 independent workstreams; confirm no shared writes.
3. **Offload evidence** — if context-offloading: record the evidence for every condition in the context-offloading gate.
4. **Profile** — discover available profiles through the tool's native mechanism; name the matching profile (scout ≠ strong model, synthesis ≠ fast model). Missing → inline.
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

Keep work inline when streams share file writes, sync cost exceeds time saved, or the work is a bounded single stream. Apply the sequential gate to dependencies.

Patterns: **fan-out/fan-in** or **scatter/gather** (N≥3 independent → wait all → parent synthesizes inline).

## Dispatch Contract (every spawn)

- Front-load all context (paths, symbols, what's known, what to return) — subagents are stateless, no mid-run questions.
- Define the return shape explicitly; vague prompts produce vague returns.
- State write vs. read-only — mismatch is the most common delegation failure.
- Parent retains all git/gh and side-effecting operations unless explicitly authorized.
- Direct the subagent to emit only its final report — no step narration or progress text in its transcript.

## Code Exploration

Read-only reconnaissance; the subagent returns a distilled brief — the parent never reads raw traversal. Specify depth and return shape (brief, file:line citations, diagram, named symbol set). Pick profile by intent: orientation vs. targeted question vs. architectural friction.

`BLOCKED`/`NO_*`/`UNAVAILABLE` → do the work inline; don't retry.

## Exploration Offloading

For exploration across files or modules, apply the dispatch rationales above. File count alone does not establish that offloading saves time or tokens.

| State | Scan shape | Action |
|---|---|---|
| Cold scan — parent doesn't hold the area | Multi-file / multi-module | Scout only when a dispatch rationale fires |
| Warm read — parent holds the bulk | A few lines, one symbol, one file | Inline — dispatch overhead exceeds savings |

Inline exceptions: exact path + line range known · a single navigation symbol lookup/reference call answers it · target file <~50 lines and isolated.
