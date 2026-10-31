---
name: session-to-memory
description: "Use at the end of a non-trivial session or when the user asks to save session learnings. Extracts retention-worthy knowledge from the session and persists it to the memory provider."
---

## Required `<project-root>/docs` reads

Shell-read before extracting or persisting session knowledge; may be gitignored — missing → note gap, never invent: `<project-root>/docs/memory-providers.md`

This skill owns bulk extraction at session end. `memory-usage` owns write triggers, naming, entry format, and in-session maintenance — defer to it for each. The memory-provider spec supplies driver mechanics (tool names, args).

## When to Run

- End of a non-trivial session that produced reusable knowledge.
- User asks to save/remember/persist the session's learnings.
- Before handing off a session that uncovered invariants, pitfalls, or decisions worth keeping.

Not for trivial sessions (single command, one-line answer, no investigation).

## Inputs

Scan: conversation history (requests, decisions, corrections, surprises) · tool output (errors, root causes, schema facts, command results) · files touched (paths, patterns, conventions) · active artifacts (plans, ADRs, commits — reference, don't duplicate).

## Extraction Targets

Pull only what clears the `memory-usage` write bar.

| Category | Keep |
| --- | --- |
| Invariants | Non-obvious constraints the code enforces |
| Conventions | Naming, layout, patterns the project mandates |
| Corrections | User corrections as the rule to follow next time |
| Preferences | How the user wants the agent to work/communicate |
| Pitfalls | Traps that required real investigation |
| Decisions | Architectural choices with lasting rationale |
| Module facts | Shape, seams, ownership, lifecycle |
| External facts | API quirks, version-specific behavior, rate limits |

Drop what fails the bar: restated code/comments, greppable style, mistake narratives, one-off phrasing/typos, reversible preferences, trivial structure, public doc defaults.

## Procedure

1. **Survey + Filter** — scan conversation history, tool outputs, touched paths, including unwritten lessons → distilled candidate list of `(topic-path, one-line substance, source citation)` clearing the write bar. Discard the rest.
2. **Dedupe** — list existing memories (`memory-usage`; `memory-providers.md`); overlapping candidate → edit existing; new topic path → new entry.
3. **Consolidate + Prune** — per `memory-usage` maintenance (merge overlaps, delete stale, repair `mem:` refs, update `project_overview`, preserve provenance + user preferences).
4. **Persist** — each write goes through the memory-provider spec's driver.
5. **Report** — one line per entry written/edited/consolidated/deleted: `topic-path — verb — source citation`; never paste memory bodies. Nothing cleared the bar → say so.

## Hard Rules

Per `memory-usage` (no conversation state, no greppable truth, no secrets, cite don't paste). Memory provider unavailable → report `BLOCKED` and stop; don't invent a fallback store.
