---
name: using-code-navigation
description: "Use code navigation tools for symbol-level work instead of grep/read: finding classes, methods, references, cross-module wiring — when tracing, navigating, or refactoring named entities (LSP-supported languages) or for onboarding/memory capture. For plain-text searches, known paths, or non-code files use grep/glob/read."
---

## Required `<project-root>/docs` reads

Shell-read before any code navigation lookup; may be gitignored — missing → note gap, never invent: `<project-root>/docs/code-navigation.md`, `<project-root>/docs/memory-providers.md`

LSP-backed semantic ops (`code-navigation.md`) — prefer over `grep`/`read` for symbol-level work.

## Project Activation (MANDATORY first step)

Every code navigation tool requires an active project — no exceptions.

1. List tools on the server once per session (`code-navigation.md`).
2. Activate project by name (folder name; full path accepted) — the response lists existing memories.

Re-activate: every new session, after `cwd` changes repos, or on any `No active project` error (pick the matching name from the error) — it's a missing-activation error, not an outage; activate and retry.

## When to Use Which Tool

| Situation | Operation | Notes |
|-----------|-----------|-------|
| First look at unfamiliar file | Symbol overview | Compact tree; `depth=1` for method list |
| Know class/method, want body | Find symbol `include_body=true` | `name_path` like `Class/method` |
| Children of class, no bodies | Find symbol `depth=1` | Cheaper than `include_body` |
| "Where is this used?" | Find referencing symbols | Needs `name_path` + `relative_path` |
| "What implements this interface?" | Find implementations | Same args as references |
| "Where is this declared?" | Find declaration | For imports/re-exports |
| Plain-text / regex search | Search for pattern | Only when symbol name unknown |
| Rename across codebase | Rename symbol | LSP-aware, updates references |
| Delete a symbol safely | Safe delete symbol | Returns refs if unsafe |
| Lint/type errors for a file | Get diagnostics | Before/after edits |
| Persist project knowledge | Write memory | Recall via read/list memories |

Exact tool names and arguments: `code-navigation.md`.

## Name Paths

`find_symbol` matches the in-file symbol tree, not the filesystem (`code-navigation.md` for details).

- `"method"` → any symbol named `method` · `"Class/method"` → suffix match · `"/Class/method"` → exact
- Overloads: `[i]` (`"Class/method[0]"`) · `substring_matching=true`: `"Foo/get"` matches `getValue`, `getData`

## Workflow Patterns

- **New file:** overview (depth 0) → find symbol `depth=1` on main class → `include_body=true` on needed methods.
- **Trace call chain:** find symbol for entry → find referencing symbols for callers → repeat until boundary (controller, route, queue).
- **Verify refactor:** find referencing symbols (empty → safe delete) → get diagnostics on touched files after edit → use rename symbol over manual edits.
- **Cross-module wiring (e.g. Laravel bindings):** find symbol on contract → find implementations → search for pattern in `ServiceProvider.php` for binding calls.

## Memories

The navigation tool may provide memory (`memory-providers.md`): per-project markdown notes, same activate-project requirement. Write/edit/delete/rename need `editing` mode — `Tool ... is not active` → add `editing` to `base_modes`/`default_modes` (`code-navigation.md`).

Operations: list memories (topic filter) · read / write / edit / delete / rename memory.

## Project Setup

- Activate project — switch by name or path · get current config — active project, available projects, modes · onboarding — once per project, generates initial memory · initial instructions — read once per session if unsure how to drive the tool.

## Limits and Fallbacks

- LSP must support the language — dynamic/eval'd symbols, Blade templates, config arrays → `grep`/`read`.
- Find referencing symbols needs `relative_path` (a file, not a directory) from a prior find symbol; external deps use `<ext...>` identifiers — never guess.
- Line numbers are 0-based — adjust against `read` output (1-based).
- `depth` before `include_body=true`; bodies only for needed methods.
- Batch independent queries — find symbol, references, implementations in parallel.
