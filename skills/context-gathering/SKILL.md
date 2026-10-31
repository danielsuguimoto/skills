---
name: context-gathering
description: "MANDATORY — invoke BEFORE any codebase exploration, code reading/writing/editing, or answering about code."
---

## Required `<project-root>/docs` reads

Shell-read before codebase exploration, code reading/writing/editing, or answering about code; may be gitignored — missing → note gap, never invent: `<project-root>/docs/code-navigation.md`, `<project-root>/docs/database-tools.md`, `<project-root>/docs/doc-lookup.md`, `<project-root>/docs/knowledge-graphs.md`

## Core Principle

Never write code from assumptions. Flow: clarify target → map (codebase + database) → fetch docs → analyze conventions → verify → act.

### 1. Clarify the target

A wrong target wastes the whole gather. Ambiguous → ask BEFORE exploring (goal, module/area, constraints); `grill-with-codebase` if complex or under-specified. Still undefined after one round → **stop and say so**, list what's ambiguous, don't explore speculatively.

### 2. Discover symbols

Symbol routing per `code-work` → *Code Reading* (`code-navigation.md`).

### 3. Map the codebase

- Knowledge graph output exists (`knowledge-graphs.md`) → query it before raw source browsing; its wiki/report covers broad navigation and architecture.
- Locate entry points, modules, dependencies; trace cross-module wiring via contracts/repositories; identify callers/callees; read similar implementations for patterns.
- Target touches persisted state → query the DB: schema, row counts, EXPLAIN plans, sample rows (`database-tools.md`). Live DB is ground truth.

### 4. Fetch current documentation

Use doc lookup tools (`doc-lookup.md`): API signatures, best practices, version-specific behavior, gotchas.

### 5. Analyze conventions

Identify: style/format rules (config files), naming, error handling/validation, testing patterns/coverage, architecture (module structure, contracts, DTOs).

### 6. Verify understanding

**Ready to act when ALL hold:** implementation located (file:line + entry points) · callers mapped (one `find_references` pass) · libraries/versions and doc patterns confirmed · conventions matched (style, naming, error handling, tests) · impact scoped (breaking changes, edge cases, error paths) · DB state verified when persisted data is touched · gaps named.

## When to skip steps

Default: don't skip — each exception is narrow, verify the condition first.

- Step 2: pure text/config with no code symbols.
- Step 3: target file fully isolated (verify first). DB queries still apply when persisted state is touched.
- Step 4: the relevant feature is verified in the last 30 lines of code already in context.
- Step 5: never skip.

## Trivial-Task Fast Path

Skip the deep scan only when ALL hold: single file with exact path known · mechanical change (rename, typo, one-line fix, add an import) · no cross-module callers (verify with one `find_references`). Then act — skip steps 2-4; step 5 still applies (match surrounding style).
