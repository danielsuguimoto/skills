---
name: memory-usage
description: >
  Use when reading or writing persistent project memory before, during, or after non-trivial work.
---

## Required `<project-root>/docs` reads

Read via shell before memory operations — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/memory-providers.md`

Persistent memory holds facts that are expensive to re-derive and survive across sessions: lessons learned while working with the user, written as short notes — not session summaries.

This skill owns when, where, and how to write, plus in-session maintenance. `session-to-memory` owns bulk extraction at session end — invoke it when a session produced several lessons or the user asks to remember the session.

## When to Use Memory

### Read before acting

| Trigger | Action |
| --- | --- |
| Starting a non-trivial task | Check the index for relevant entries. |
| Unfamiliar error, pattern, or convention | Search memories for a matching topic or pitfall. |
| User asks "how do we do X here?" | Read the related memory first. |

### Write after learning

| Trigger | Action |
| --- | --- |
| Non-obvious invariant or convention | Write it under a topic-specific name. |
| Tricky pitfall that required real investigation | Write it under a `pitfalls/` path. |
| Architectural decision with lasting rationale | Write it under a `decisions/` path. |
| User corrects you or states a preference | Write the correction plus the rule to follow next time. |
| User explicitly asks to remember something | Write it with the requested name. |

Write at the moment of learning — a correction, preference, or hard-won lesson is freshest in context. Defer only bulk review to `session-to-memory`.

### Do NOT write

- Conversation state, task progress, session summaries, or scratchpads — memory stores lessons, not history.
- Transient data (current branch, in-flight PR numbers, today's ticket ID).
- Secrets, credentials, or PII.
- Duplicates of an existing memory. Edit the existing one instead.

## Entry Format

- Short one-line bullets under topic headings. Carry provenance per entry: `— [source: <session or task ref>; added: YYYY-MM-DD]`.
- `project_overview` doubles as the index: general preferences plus `mem:` pointers to topic entries. Keep it current when adding, renaming, or deleting entries.
- Read the index first, then load topic entries on demand — never dump the whole store into context.

## Maintain As You Go

Memory work includes improving the store, not only appending. Whenever you read or write memory:

- Consolidate overlapping entries into the better-named one.
- Delete stale entries — transient details, reversed decisions, notes no session would use.
- Repair broken `mem:` references and stale index pointers.
- Prefer edit over delete-and-recreate to keep `mem:` references stable.
- Preserve provenance and explicit user preferences when consolidating or pruning — never drop them during cleanup.
- Boundary check: a lesson that is a repeatable procedure belongs in a skill, not memory.

## Core Rules

- Read before non-trivial work.
- Never duplicate codebase-readable truth. If a quick `grep` or `read` recovers it, leave it in code.
- Keep memories structured with headings, bullets, and tables. Split entries past ~100 lines.
- Cross-reference with `mem:<name>` — cite, don't paste bodies.
- A memory written this session applies to future sessions — act on what you learned now; don't rely on re-reading it.

## Memory Naming Conventions

Use slash-separated topic paths. Be specific, not generic.

- `project_overview` — high-level project shape and constraints.
- `modules/<name>` — module-specific facts, architecture, or relationships.
- `modules/<name>/<subtopic>` — narrow subtopics (e.g., lifecycle, repositories).
- `pitfalls/<topic>` — non-obvious traps and how to avoid them.
- `decisions/<topic>` — architectural decisions and their rationale.
- `global/<topic>` — cross-project knowledge only if it truly applies everywhere.

## Generic Memory Operations

All memory drivers expose roughly the same lifecycle (`<project-root>/docs/memory-providers.md` for exact tool names and arguments).

| Operation | Purpose |
| --- | --- |
| List | See what memories exist, optionally filtered by topic. |
| Read | Load a specific memory into context. |
| Write | Create a new memory entry. |
| Edit | Update an existing memory while keeping its name stable. |
| Delete | Remove obsolete or wrong memories. |
| Rename | Fix a memory name without breaking references. |
| Activate project | Scope the driver to the correct project before any memory call. |

## Project Activation

Many memory drivers require an active project before listing or reading. Verify the active project before memory operations. If the driver reports no active project, activate the correct one.

