---
name: context-gathering
description: MANDATORY HOW-skill. Must be invoked BEFORE any codebase exploration, code reading, writing, editing, or answering about code. If this skill is not active in the current conversation, DO NOT explore/read/edit code or answer code questions.
---

## Core Principle

Never write code from assumptions. Gather context first, then act.

Flow: clarify target → delegate to a subagent (non-trivial) or inline → map (codebase + database) → fetch docs (delegate to a subagent) → analyze conventions → verify → act.

## Pre-flight checkpoint

Before any codebase exploration, `read`/`edit`/`write` on a code file, any Serena `find_symbol`/`find_references` call, or any answer about code, verify:
1. The target (file path, symbol, or question) is defined — if ambiguous, ask before exploring.
2. If a dispatch gate fires for steps 2-5, delegate instead of gathering inline.

## Delegation

Delegate steps 2-5 to a subagent, step 4 to a doc-fetching subagent. Inline only on BLOCKED. Parent keeps: clarify (step 1), verify (step 6), plan, edit.

| Step | Delegate when… | Keep inline when… |
|---|---|---|
| 2 Discover symbols | Search spans multiple modules or returns many candidates | One or two known symbols, single `find_symbol` call |
| 3 Map codebase | Always delegate on non-trivial code tasks (gate fires) | Target file fully isolated and small (rare) |
| 3a Query database | Schema/counts/EXPLAIN span multiple tables or inform a data-driven bug | Single-table lookup with a known query already in context |
| 4 Fetch docs | Always delegate — any framework/library/SDK lookup is unbounded cost | Feature verified in last 30 lines of code already in context |
| 5 Analyze conventions | Convention sources spread across many files or unfamiliar area | Conventions already in module `AGENTS.md` in context |

If unsure whether volume is bounded, delegate.

### 1. Clarify the target

A wrong target wastes the whole gather. If the request is ambiguous, ask BEFORE exploring: specific goal, affected module/area, constraints. Use `grill-me` if complex or under-specified. If the target is still undefined after one clarifying round, **stop and say so explicitly.** List what's ambiguous. Do not explore speculatively.

### 2. Discover symbols (Serena-first)

Serena MCP is PRIMARY for symbol-level work: `find_symbol`, `find_references`, list class members/signatures/parents/interfaces. Use `grep`/`read`/`find_file_by_name` ONLY for non-symbol lookups: plain-text searches (config keys, lang keys, strings), known file paths, glob patterns.

### 3. Map the codebase

- exlink only: if `graphify-out/graph.json` exists, query it before raw source browsing (`graphify query "<question>"`, `graphify path "<A>" "<B>"`, `graphify explain "<concept>"`). Fallback to `graphify-out/wiki/index.md` for broad navigation, `graphify-out/GRAPH_REPORT.md` for architecture review.
- Locate entry points, related modules, dependencies. Read module-specific `AGENTS.md` files for conventions. Trace cross-module wiring via contracts and repositories. Identify callers and callees. Read existing implementations of similar features for patterns.
- Query the database when the target touches persisted state: schema, row counts, EXPLAIN plans, sample rows. Use Laravel Boost MCP database tools and Tinker REPL. Database state is ground truth — verify it alongside code. Do not rely on seeders, migrations, factories, or operations for current data shape.

### 4. Fetch current documentation

Assume internal knowledge is outdated for any framework/library/SDK. Use Context7 MCP, then Laravel Boost MCP for Laravel-specific docs, then Tavily/Brave as fallback. Verify API signatures, current best practices, version-specific behavior, and gotchas.

### 5. Analyze conventions

Before proposing changes, identify: code style and formatting rules (e.g., `.pint.json`), naming patterns, error handling and validation patterns, testing patterns and existing coverage, architecture patterns (module structure, repository/contract, DTOs). Never skip this step.

### 6. Verify understanding

Steps 2-5 collect material; step 6 confirms it is enough to act. Skipping leads to gaps that surface as rework or broken callers.

Confirm you know: current implementation and callers, libraries/versions, relevant doc patterns, impact areas and breaking changes, edge cases and error paths, and database state (schema, counts, sample rows) when the target touches persisted data.

**Ready to act when ALL hold:** implementation located (file:line + entry points named) · callers mapped (one `find_references` pass) · conventions matched (style, naming, error handling, test patterns) · impact scoped (breaking changes, edge cases, error paths) · gaps named (if any above can't be confirmed, state what's unknown and what's needed).

Only after this — plan and implement.

## When to skip steps

Default: don't skip. Each exception is narrow — verify the condition holds first.

- Step 2: only for pure text/config with no code symbols.
- Step 3: only if the target file is fully isolated (verify first). Database queries still apply if the target touches persisted state.
- Step 4: only if the relevant feature is verified in the last 30 lines of code already in context.
- Step 5: never skip.

## Trivial-Task Fast Path

Skip the deep scan only when ALL hold: single file with exact path known (no search needed), change is mechanical/obvious (rename, typo, one-line fix, add an import), no cross-module callers affected (verify with one `find_references` call).

When the fast path applies: act. Skip steps 2-4. Step 5 (conventions) still applies — match surrounding style. If any condition fails, run the full workflow.
