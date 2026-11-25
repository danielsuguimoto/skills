---
name: subagents
description: "Delegation gates. MANDATORY before any spawn, any large context-gathering scan (codebase, DB, docs, memories), any work splittable into parallel independent tasks, any bounded task a cheaper model tier could run, or any handoff to another agent CLI."
---

Default to execution in the current session. Delegate a workflow or its steps only when a dispatch rationale below fires or the user explicitly asks to delegate.

**Ticket Brief exception.** When a workflow calls for a `Ticket Brief`, delegate only that isolated, read-only ticket inspection — unless a matching brief or ticket content covering the brief's evidence depth (comments and attachments included) is already in session, in which case derive the scoped answers inline. Follow its isolation and blocker rules instead of the sequential, size, and inline-fallback gates below. The parent runs the workflow and never probes raw ticket content to qualify.

## Sequential Gate

Keep sequential work in the parent. If B needs A's result, run A then B inline. This gate applies to one blocking handoff as well as a chain of subagents; waiting for a child is not parallel work. Fanning out independent units inside one step is not a sequential handoff — it reaches the Dispatch Rationales directly.

Three things cross this gate: an explicit user request to delegate that work, a model-tier offload whose conditions all hold, or a read-only external handoff for a documented capability the parent lacks. Context offloading, role specialization, and foreground execution do not. Apply this gate before the other dispatch rationales; evaluating a model-tier candidate includes its profile lookup.

## Dispatch Rationales

After the sequential gate passes, spawn **only** when at least one holds — the gated rationales are strict; when in doubt, stay inline.

| Rationale | When to delegate |
|-----------|------------------|
| Parallel time win | **≥3 independent workstreams**, no shared writes. Below 3, coordination overhead eats the gain. |
| Context offloading | Exceptional read-only research that passes the context-offloading gate below. |
| Model-tier offload | Bounded, low-reasoning work that passes the model-tier gate below — may override a sequential dependency. |
| External capability | Read-only external agent handoff for a documented capability the parent lacks — conditions in External Agent Handoff below. |
| Explicit user request | User asks to spawn/delegate/parallelize that work. A request to complete a task or use a skill is not delegation permission. |

### Context-Offloading Gate

Start inline with path listings, targeted searches, and bounded reads. A folder with 10 files is ordinary inline work; file count, repository size, unfamiliarity, and multiple modules do not justify delegation.

Offload only when those probes establish **all** of the following:

- The question requires inspecting **more than 10,000 tokens** of still-unread, task-relevant material. Name the sources and give a size estimate grounded in observed output; count relevant sections, not whole folders or unrelated file contents.
- Narrower searches or reads cannot answer the question. State why.
- The parent needs only a cited brief and will not reread the raw material.

Unknown size or uncertain necessity → stay inline. The size floor is necessary, not permission by itself; the sequential gate still applies.

### Model-Tier Gate

Delegate for cost only when **all** hold:

- **Bounded** — every input fits the dispatch prompt; no mid-run dependence on parent context.
- **Low reasoning demand** — mechanical transforms, bulk repetitive edits, boilerplate, extraction, test/lint runs. Design, debugging, and judgment calls stay at parent tier.
- **A cheaper-tier profile exists** — check `<project-root>/docs/models.md` first, else the tool's native mechanism; its allowed tools cover the task's required operations (read vs. write). None → inline.
- **Delegation pays** — the win comes from volume: many files, long outputs, or repeated invocations. A few-line edit or single lookup never clears spawn overhead.

## Pre-Spawn Checklist (fill inline before every subagent dispatch)

1. **Rationale** — which gate fires? State it.
2. **Parallelism** — if parallel-time-win: list the ≥3 independent workstreams; confirm no shared writes.
3. **Offload evidence** — if context-offloading or model-tier: record the evidence for every condition in that rationale's gate.
4. **Profile** — discover available profiles through the tool's native mechanism; name the matching profile (scout ≠ strong model, synthesis ≠ fast model). Missing → inline.
5. **Foreground/background** — foreground default; background only if all background conditions hold (state them).

Can't fill all five → do the work inline.

## Do NOT spawn for

- Anything answerable from context already in this session — unless it clears the model-tier gate.
- A single file edit, symbol lookup, or known path + line range.
- A bounded next step where the parent holds all inputs — unless it clears the model-tier gate.
- Avoiding a thinking step.

## Foreground Preference

Background subagents have reduced capabilities: unapproved tools auto-denied, `exec` unavailable, no new permission prompts — they fail silently on approval-gated tasks. Use background only when ALL hold: ≥3 independent workstreams · needed tools pre-approved or read-only (grep, glob, read) · parent has productive work meanwhile. Otherwise foreground.

## Model Routing

Route by task shape — strong models cost latency and tokens; weak models fail complex reasoning.

| Task shape | Tier |
|---|---|
| Code exploration, scout brief, symbol trace | Fast / cheap |
| Mechanical edits, boilerplate, extraction | Fast / cheap |
| Doc fetch, library/SDK lookup | Mid |
| Implementation on disjoint files, PR-fix, synthesis | Strong |
| Memory extraction, to-specs planning | Mid–strong |
| Test suite execution | Fast |

Tier comes from the profile, not a model parameter — pick the profile configured for the tier. Where the harness supports per-profile model pins, a custom profile pinning a cheaper model is the only route to a write-capable cheap subagent. No cheaper profile → run the task inline at parent tier. The tier→model mapping and the profile-pin mechanism live in `<project-root>/docs/models.md`; missing spec → discover via the harness's native mechanism.

**Defaults:** auto-routed → trust it, override only on clear mismatch; unsupported → inline or accept inheritance (never spawn a second subagent to work around it).

## Parallel Execution

Keep work inline when streams share file writes, sync cost exceeds time saved, or the work is a bounded single stream. Apply the sequential gate to dependencies.

Shared infrastructure is not the barrier: one database, server, or browser process qualifies when streams use disjoint actors and records, write only records and files assigned to their actor or tenant, and avoid writes to shared mutable state — tabs in one browser context share cookies and storage, so each parallel actor needs a separate context (or profile where the tool has no context support). Doubtful independence stays inline. Before retry, the parent removes or quarantines any partial state a failed stream wrote so siblings are not contaminated.

Patterns: **fan-out/fan-in** or **scatter/gather** (N≥3 independent → wait all → parent synthesizes inline).

For parallel fan-out, use the active ledger — a native todo list, workflow plan, or inline outcome sequence:

- Before dispatch, the parent projects one item per workstream into that ledger and owns every status transition.
- Assign each workstream a stable stream ID (`W1`, `W2`, …). Use it in the ledger item and dispatch prompt so fan-in results map without inference.
- Require the final report's first line as `<stream-id>: <status> — <one-line result>`, with `<status>` = `completed`, `blocked`, or `failed`; the dispatch's return shape supplies the remainder.

## Dispatch Contract (every spawn)

- Front-load all context (paths, symbols, what's known, what to return) — subagents are stateless, no mid-run questions.
- Define the return shape explicitly; vague prompts produce vague returns.
- State write vs. read-only — mismatch is the most common delegation failure.
- Parent retains all git/gh and side-effecting operations unless explicitly authorized.
- Direct the subagent to emit only its final report — no step narration or progress text in its transcript.

## Code Exploration

Read-only reconnaissance; the subagent returns a distilled brief — the parent never reads raw traversal. Specify depth and return shape (brief, file:line citations, diagram, named symbol set). Pick profile by intent: orientation vs. targeted question vs. architectural friction.

`BLOCKED`/`NO_*`/`UNAVAILABLE` → do the work inline; don't retry.

## External Agent Handoff

Other agent harnesses run through their own CLIs, not the subagent mechanism — see `<project-root>/docs/external-agents.md` for installed harnesses, headless invocation, and output capture. Missing spec → ask to configure it; do not invent invocations from `--help`.

Delegate externally only on explicit user request, or when a documented capability the parent lacks is required — name it. Write-capable external handoffs always require explicit request. The dispatch contract applies unchanged: front-load context, define the return shape, state write vs. read-only; scope the prompt tightly (file boundaries, no git/gh unless explicitly authorized). Parent retains git/gh.

On missing binary, non-zero exit, or unreadable output, stop and report; do not silently retry. External handoffs are synchronous by default; async modes are only allowed if `docs/external-agents.md` documents how to collect the result.

## Exploration Offloading

For exploration across files or modules, apply the dispatch rationales above. File count alone does not establish that offloading saves time or tokens.

| State | Scan shape | Action |
|---|---|---|
| Cold scan — parent doesn't hold the area | Multi-file / multi-module | Scout only when a dispatch rationale fires |
| Warm read — parent holds the bulk | A few lines, one symbol, one file | Inline — dispatch overhead exceeds savings |

Inline exceptions: exact path + line range known · a single navigation symbol lookup/reference call answers it · target file <~50 lines and isolated.
