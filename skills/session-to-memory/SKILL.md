---
name: session-to-memory
description: >
  Use at the end of a non-trivial session or when the user asks to
  "remember this session" or "save what we learned". Extracts
  retention-worthy knowledge from the current session's conversation history
  and context, then persists it to the active memory provider.
---

## Required `<project-root>/docs` reads

Read via shell before extracting or persisting session knowledge — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/memory-providers.md`

This skill owns bulk extraction at session end. `memory-usage` owns write triggers, naming, entry format, and in-session maintenance — defer to it for each. The memory-provider spec supplies driver mechanics (tool names, args).

## When to Run

- End of a non-trivial session that produced reusable knowledge.
- User asks to save/remember/persist the session's learnings.
- Before `handoff` when the session uncovered invariants, pitfalls, or decisions worth keeping.

Not for trivial sessions (single command, one-line answer, no investigation).

## Inputs

| Source | What to scan |
| --- | --- |
| Conversation history | User requests, decisions, corrections, surprises. |
| Tool output | Errors, root causes, schema facts, command results. |
| Files touched | Paths, patterns, conventions discovered. |
| Active artifacts | Plans, ADRs, PRDs, commits — reference, don't duplicate. |

## Extraction Targets

Pull only what clears the `memory-usage` write bar: reusable, hard-won, non-obvious, not trivially greppable.

| Category | Keep | Drop |
| --- | --- | --- |
| Invariants | Non-obvious constraints the code enforces | Comments that restate code |
| Conventions | Naming, layout, patterns the project mandates | Style recoverable from one file |
| Corrections | User corrections as the rule to follow next time | The mistake narrative |
| Preferences | How the user wants the agent to work/communicate | One-off phrasing choices |
| Pitfalls | Traps that required real investigation | One-off typos |
| Decisions | Architectural choices with lasting rationale | Reversible preferences |
| Module facts | Shape, seams, ownership, lifecycle | Trivial structure |
| External facts | API quirks, version-specific behavior, rate limits | Public doc defaults |

## Procedure

1. **Survey + Filter** — scan conversation history, tool outputs, touched paths, including unwritten lessons → distilled candidate list of `(topic-path, one-line substance, source citation)` clearing the write bar. Discard the rest.
2. **Dedupe** — list existing memories (`memory-usage`; `<project-root>/docs/memory-providers.md`); overlapping candidate → edit existing; new topic path → new entry.
3. **Consolidate + Prune** — leave the store better: merge overlaps, delete stale/transient, repair `mem:` references, update `project_overview`. Preserve provenance and explicit user preferences.
4. **Persist** — each write goes through the memory-provider spec's driver.
5. **Report** — one line per entry written/edited/consolidated/deleted: `topic-path — verb — source citation`. Nothing cleared the bar → say so.

## Output Contract

Compact changelog of entries written/edited/consolidated/deleted with citations — never paste memory bodies.

## Hard Rules

- Never persist conversation state, task progress, or scratchpads.
- Never duplicate codebase-readable truth — cite file:line instead.
- Never paste full diffs, logs, or file contents into memory — distill.
- Never write secrets, credentials, or PII.
- Memory provider unavailable → report `BLOCKED` and stop; don't invent a fallback store.
