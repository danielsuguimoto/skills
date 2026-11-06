---
name: context-gathering
description: "MANDATORY — invoke BEFORE any codebase exploration, code reading/writing/editing, or answering about code."
---

## Tool specs

Shell-read `<project-root>/docs/<category>.md` before first use of that category — `code-navigation.md` for symbol lookups/references · `database-tools.md` when the task touches persisted state · `knowledge-graphs.md` when graph output exists · `doc-lookup.md` before fetching external docs. May be gitignored — missing → note gap, never invent.

## Core Principle

Never write code from assumptions — gather the minimum that answers the questions the change raises, then stop. Context past them is load: it thins attention and steers solutions toward irrelevant code.

Flow: frame the questions → pull answers → verify → act.

### 1. Frame the questions

A wrong target wastes the whole gather. Name the 2–4 things the change must know — e.g. where does X get called · what shape does Y return · which convention does this neighborhood follow · does Z persist.

Ambiguous → ask BEFORE exploring (goal, module/area, constraints); `grill-with-codebase` if complex or under-specified. Still undefined after one round → **stop and say so**, listing what's ambiguous.

### 2. Pull answers

Work the questions one at a time; each has a target and is done when answered.

- **Symbols** — routing per `code-work` → *Code Reading* (`code-navigation.md`). Locations and signatures first; bodies only for code you'll change or imitate.
- **The slice** — follow the change's path: entry point → target → direct callers/collaborators. For a pattern, read the closest similar implementation — one, not a survey. Broad architecture question + graph output exists (`knowledge-graphs.md`) → query the graph instead of browsing source.
- **Persisted state** — only when touched: schema, row counts, sample rows, EXPLAIN plans for the queries involved (`database-tools.md`). Live DB is ground truth.
- **External docs** — only when a library API/version is genuinely uncertain (`doc-lookup.md`); code already in context answers first.

Widen on evidence — a reference that surprises, an invariant the slice can't explain, a new question an answer raised. Answer it, then return to the list.

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

The gather is done when the framed questions are answered — stop there. Whole modules, sibling features, architecture tours read "for context" — load, not signal.
