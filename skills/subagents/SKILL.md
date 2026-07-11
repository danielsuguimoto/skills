---
name: subagents
description: >
  Subagent spawning rules. Activate when considering spawning a subagent for
  parallel work, isolated tasks, independent execution, code exploration, or
  context offloading.
---

Discover profiles through your tool's native mechanism. Don't assume a fixed list or path. Apply rationales, gates, and contracts by judgment. The listed situations are not exhaustive.

## Dispatch Rationales

| Rationale | When to delegate |
|-----------|------------------|
| Context offloading | Unbounded-cost work (docs, memory bodies, Q&A transcripts) |
| Parallel execution | ≥2 independent workstreams, no shared writes |
| Independent execution | Self-contained task, no parent intervention needed |
| Code exploration | Read-only reconnaissance, distilled brief sufficient |

**Default to inline** when work is bounded and decision-ready. Delegate when any rationale produces net win. Gate fires → delegate. No gate + bounded + decision-ready → inline. No gate + unbounded → delegate. Independent workstreams ≥2 → parallel dispatch.

## Precedence Rule (MANDATORY)

Before executing any WHAT-skill's inline work, check the dispatch gates below. If a gate fires, delegate instead of inline. This skill decides where work runs; the WHAT-skill defines what work is done.

Gates fire on **task shape** (observable from the request and the WHAT-skill's trigger), not on predicted work volume. This avoids the chicken-and-egg of predicting file counts before reading files.

| WHAT-skill trigger | Gate (fires when…) | Rationale |
|---|---|---|
| Context-gathering step 3 "Map the codebase" | non-trivial code task (step 3 is inherently multi-file) | offloading + exploration |
| Context-gathering step 4 / repo-reconnaissance step 6 (doc fetch) | any framework/library/SDK question (always delegate) | offloading |
| Memory-usage read trigger | any read trigger fires (always delegate) | offloading |
| Grill-me (ambiguous/under-specified plan) | Grill-me would trigger (foreground) | offloading |
| Triage debug-mode investigation | ticket issue with codebase/DB/log/doc investigation (always delegate) | offloading |
| Triage ask-mode answer | answer needs codebase or library/SDK context (always delegate) | offloading |
| To-specs planning | non-trivial plan (always delegate) | offloading + exploration |
| Implement kickoff | ticket implementation kickoff (always delegate) | offloading + exploration |
| Session-to-memory extraction | extraction fires (always delegate) | offloading |
| Code-work edit | edit spans >1 module or >3 file reads (Mandatory Offloading Threshold) | offloading |
| PR-fix feedback analysis (Steps 2, 4, 5) | PR with review feedback (always delegate) | offloading |
| PR-create / ship-changes metadata generation | PR or ticket title/body/checklist from a diff (always delegate) | offloading |
| Validation step running test suites | any test invocation in a WHAT-skill validation step | offloading |
| Multi-subagent synthesis | ≥2 sibling subagents returned verbose blocks | offloading |
| Read-only codebase question | spans multiple files/modules or area is unfamiliar | exploration + offloading |
| Self-contained implementation on disjoint files | clear boundaries, no shared file writes | independent execution |
| Multi-step task with N independent workstreams (N ≥ 2) | no shared file writes, no cross-stream coordination | parallel + independent |

## Foreground Preference

**Default to foreground.** Background subagents have reduced capabilities: unapproved tools are auto-denied, `exec` is unavailable, and the subagent cannot prompt for new permissions. This means background subagents often fail silently on tasks requiring tool approval.

Use background **only** when all of the following hold:
- ≥2 truly independent workstreams benefit from parallelism
- The subagent's needed tools are already pre-approved or read-only (grep, glob, read)
- The parent has productive work to do while waiting

Otherwise spawn foreground. The parent pauses but the subagent gets full tool access and can prompt for permissions.

Profiles are discovered dynamically. Verify availability before delegating. If missing, fall back to inline per the WHAT-skill.

## Parallel Execution

Spawn in parallel when workstreams are independent (no shared file writes, no cross-stream data dependency). **Don't parallelize** when: workstreams share file writes (serialize/merge), one stream feeds the next (serialize/fan-in), sync cost exceeds time saved (inline), or the work is a bounded single stream (inline is cheaper).

Patterns: **Fan-out / fan-in** (N independent → wait all → optionally a synthesis subagent), **Pipeline** (A's output feeds B → serialize), **Scatter / gather** (N subagents, parent synthesizes inline when results are small).

## Independent Execution

Self-contained task runs to completion without parent intervention. Independence contract:
- Front-load all context (file paths, function names, what's known, what to return). Subagents are stateless — no mid-run clarifying questions.
- Define return shape explicitly. Vague prompts produce vague returns.
- State write vs. read-only. Mismatch is the most common delegation failure.
- Parent retains all git/gh and side-effecting operations unless explicitly authorized.

## Code Exploration

Read-only reconnaissance. Subagent returns a distilled brief; parent never reads raw traversal. Specify depth and return shape (brief, file:line citations, diagram, named symbol set). Pick profile by intent: orientation vs. targeted question vs. architectural friction.

Fallback on `BLOCKED`: if a subagent returns `BLOCKED` / `NO_*` / `UNAVAILABLE`, do that work inline per the WHAT-skill. Don't retry the subagent.

## Mandatory Offloading Threshold

Hard gate (overrides "default to inline"): if a task needs **more than 3 file reads** OR spans **more than 1 module** for exploration/tracing, dispatch a read-only scout BEFORE any inline `read`/`grep`/code navigation query (see `/docs/code-navigation.md` in the project root). The parent acts on the brief; raw file contents never enter parent context.

Rationale: exploration tokens are the largest avoidable parent-context cost. Once read, those tokens stay in context for the whole session. A scout's distilled brief costs a fraction and is discardable.

Exceptions (inline allowed): exact file path and line range already known; a single code navigation symbol lookup / reference trace call (see `/docs/code-navigation.md` in the project root) answers it; target file under ~50 lines and fully isolated.

## Failure Modes and Red Flags

- **Premature completion** — parent skips gate check to "just do it inline". Gate check is one table lookup. Sharpen the gate match first; only if no row fires is inline correct.
- **Vague return shape** — subagent dumps raw content. State the return shape in the prompt before dispatch. "Distilled brief with file:line citations", not "look at the code".
- **Read-only / write mismatch** — subagent edits when it should read, or vice versa. State read-only vs. write in every dispatch prompt. Most common delegation failure.
- **Retry loop** — parent re-dispatches a subagent that returned `BLOCKED`. Do the work inline. Don't retry.

**Never:** delegate without stating read-only vs. write; retry a `BLOCKED` subagent; let a scout dump raw file contents; parallelize workstreams that share file writes.

**Always:** front-load all context (subagents are stateless); check dispatch gates before inline WHAT-skill work; parent retains all git/gh and side-effecting operations; delegate unbounded-cost work (docs, memory dumps, Q&A transcripts).

