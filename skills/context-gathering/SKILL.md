---
name: context-gathering
description: "Required before exploring, reading, writing, editing, or answering about code."
---

## Tool specs

Shell-read `<project-root>/docs/<category>.md` before first use of that category — `code-navigation.md` for symbol lookups/references · `database-tools.md` when the task touches persisted state · `knowledge-graphs.md` when graph output exists · `doc-lookup.md` before fetching external docs. May be gitignored — missing → note gap, never invent.

## Core Principle

Gather the minimum evidence needed for the change, then stop; never write code from assumptions.

### 1. Frame the questions

Name the 2–4 questions the change must answer.

Ambiguous → ask BEFORE exploring (goal, module/area, constraints); `grill-with-codebase` if complex or under-specified. Still undefined after one round → **stop and say so**, listing what's ambiguous.

### 2. Pull answers

Give each question a target and stop when it is answered. Batch independent lookups.

- **Symbols** — routing per `code-work` → *Code Reading* (`code-navigation.md`). Locations and signatures first; bodies only for code you'll change or imitate.
- **The slice** — follow the change's path: entry point → target → direct callers/collaborators. For a pattern, read the closest similar implementation — one, not a survey. Broad architecture question + graph output exists (`knowledge-graphs.md`) → query the graph instead of browsing source.
- **Persisted state** — only when touched: schema, row counts, sample rows, EXPLAIN plans for the queries involved (`database-tools.md`). Live DB is ground truth.
- **External docs** — only when a library API/version is genuinely uncertain (`doc-lookup.md`); code already in context answers first.

Widen when evidence raises an unanswered question, then return to the list. Non-trivial widening runs as sequential thinking.

### 3. Verify — ready to act

Ready when every framed question has an answer grounded in code or docs read this session:

- target located (file:line + entry point)
- callers of touched symbols traced (one `find_references` pass each)
- conventions matched from the code this change sits beside (style, naming, error handling, tests)
- unknowns named — no assumptions silently carried

A criterion that doesn't apply → skip it explicitly. The gate binds the framed questions, not a fixed checklist.

## Depth scales with blast radius

| Change | Gather |
|---|---|
| Mechanical — exact path known, single file, no cross-module callers | Read the target, match surrounding style, act. If anything crosses modules, verify callers with one `find_references`. |
| Behavioral fix / additive feature | Full pull on the slice. |
| Contract, signature, or schema change | Slice + blast radius: every caller and implementer, every consumer of the changed shape. |

Stop when the framed questions are answered; avoid unrelated modules and architecture tours.
