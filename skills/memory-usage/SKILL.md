---
name: memory-usage
description: "Use for persistent project memory reads/writes during non-trivial work."
---

## Required `<project-root>/docs` reads

Shell-read before memory operations; may be gitignored — missing → note gap, never invent: `<project-root>/docs/memory-providers.md`

Persistent memory holds facts that are expensive to re-derive and survive across sessions: lessons learned working with the user, written as short notes — not session summaries.

This skill owns when, where, and how to write, plus in-session maintenance. `session-to-memory` owns bulk extraction at session end — invoke it after a session produced several lessons or the user asks to remember the session.

## When to Use Memory

- Starting a non-trivial task → check the index for relevant entries.
- Unfamiliar error, pattern, or convention → search memories for a matching topic or pitfall.
- User asks "how do we do X here?" → read the related memory first.
- Non-obvious invariant or convention learned → write under a topic-specific name.
- Tricky pitfall requiring real investigation → `pitfalls/` path; architectural decision with lasting rationale → `decisions/` path.
- User corrects you or states a preference → write the correction plus the rule to follow next time.
- User asks to remember something → write it with the requested name.

Write at the moment of learning — defer only bulk review to `session-to-memory`.

### Do NOT write

- Conversation state, task progress, session summaries, scratchpads.
- Transient data (current branch, in-flight PR numbers, today's ticket ID).
- Secrets, credentials, or PII.
- Duplicates — edit the existing entry instead.

## Entry Format

- Short one-line bullets under topic headings, provenance per entry: `— [source: <session or task ref>; added: YYYY-MM-DD]`.
- `project_overview` doubles as the index: general preferences plus `mem:` pointers to topics. Keep it current on add/rename/delete.
- Read the index first, then load topics on demand.

## Maintain As You Go

Whenever you read or write memory:

- Consolidate overlapping entries into the better-named one.
- Delete stale entries (transient details, reversed decisions, notes no session would use).
- Repair broken `mem:` references and stale index pointers.
- Prefer edit over delete-and-recreate — keeps `mem:` references stable.
- Preserve provenance and explicit user preferences when consolidating or pruning.
- A lesson that is a repeatable procedure belongs in a skill, not memory.

## Core Rules

- Never duplicate codebase-readable truth — a quick `grep`/`read` recovers it, leave it in code.
- Structure with headings, bullets, tables; split entries past ~100 lines.
- Cross-reference with `mem:<name>` — cite, don't paste bodies.
- A memory written this session applies to future sessions — act on it now.

## Naming Conventions

Slash-separated topic paths; specific, not generic.

- `project_overview` — project shape and constraints; the index.
- `modules/<name>[/<subtopic>]` — module facts, architecture, relationships, narrow subtopics.
- `pitfalls/<topic>` — non-obvious traps and avoidance.
- `decisions/<topic>` — architectural decisions and rationale.
- `global/<topic>` — truly cross-project knowledge only.

## Operations

All memory drivers expose the same lifecycle — exact tool names and args in `memory-providers.md`: list (optional topic filter), read, write, edit (name-stable), delete, rename, activate-project. Many drivers require an active project — verify before listing or reading; activate the correct one if none.
