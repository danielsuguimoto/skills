---
name: using-serena
description: >
  Use Serena MCP for symbol-level code work instead of grep/read: finding classes,
  methods, functions, references, and cross-module wiring. Activate when tracing,
  navigating, or refactoring named code entities in an LSP-supported language, or
  for project onboarding/memory capture. Do NOT activate for plain-text searches,
  known file paths, or non-code files — use grep/glob/read there.
---

Serena provides LSP-backed semantic tools. Prefer them over `grep`/`read` for symbol-level work; they are token-efficient and accurate.

## Project Activation (MANDATORY first step)

Every Serena tool requires an active project. No exceptions.

1. Call `mcp_list_tools` on `serena` once per session (not per call).
2. Call `activate_project` with the project name (folder name, not full path). For this workspace: `{"project": "exlink"}`. Accepts full path too if unsure.
3. Read the response — it lists existing memories.

Re-activate: start of every new session, after `cwd` changes to a different repo, or if any tool returns `No active project` (pick the matching name from the error). Treating `No active project` as a server outage is a missing-activation error — activate and retry.

## When to Use Which Tool

| Situation | Tool | Notes |
|-----------|------|-------|
| First look at unfamiliar file | `get_symbols_overview` | Compact symbol tree; `depth=1` for method list |
| Know class/method name, want body | `find_symbol` with `include_body=true` | Use `name_path` like `Class/method` |
| Children of class without bodies | `find_symbol` with `depth=1` | Cheaper than `include_body` |
| "Where is this used?" | `find_referencing_symbols` | Requires `name_path` + `relative_path` |
| "What implements this interface?" | `find_implementations` | Same args as references |
| "Where is this declared?" | `find_declaration` | For imports/re-exports |
| Plain-text / regex search | `search_for_pattern` | Only when symbol name unknown |
| Rename across codebase | `rename_symbol` | LSP-aware, updates all references |
| Delete a symbol safely | `safe_delete_symbol` | Returns refs if unsafe |
| Lint/type errors for a file | `get_diagnostics_for_file` | Use before/after edits |
| Persist project knowledge | `write_memory` | Recall with `read_memory` / `list_memories` |

## Name Paths

`find_symbol` matches against the in-file symbol tree, not the filesystem.

- Simple: `"method"` → any symbol named `method`
- Relative: `"Class/method"` → matches suffix
- Absolute: `"/Class/method"` → exact match only
- Overloads: append `[i]` (e.g. `"Class/method[0]"`)
- Substring on last element: `substring_matching=true` makes `"Foo/get"` match `getValue`, `getData`

## Workflow Patterns

**Understanding a new file:** `get_symbols_overview` (depth 0) → `find_symbol` with `depth=1` on main class → `find_symbol` with `include_body=true` on specific method(s).

**Tracing a call chain:** `find_symbol` to locate entry point → `find_referencing_symbols` to find callers → repeat on each caller until boundary (controller, route, queue).

**Verifying a refactor:** `find_referencing_symbols` on the symbol → if empty, `safe_delete_symbol` is safe → after edit, `get_diagnostics_for_file` on touched files → for renames, use `rename_symbol` instead of manual edits.

**Cross-module wiring (e.g. Laravel bindings):** `find_symbol` on the contract → `find_implementations` for concrete classes → `search_for_pattern` in `ServiceProvider.php` files for binding calls.

## Memories

Serena is the primary memory driver. Memories are persistent, per-project markdown notes under `.serena/memories/`.

Memory tools require an active project (same `activate_project` call). Write tools (`write_memory`, `edit_memory`, `delete_memory`, `rename_memory`) require `editing` mode — if they return `Tool ... is not active`, the Serena config (`~/.serena/serena_config.yml`) must include `editing` in `base_modes`/`default_modes`.

| Operation | Tool |
| --- | --- |
| List (optionally by topic) | `list_memories` |
| Read / Write new / Update / Delete / Rename | `read_memory` / `write_memory` / `edit_memory` / `delete_memory` / `rename_memory` |

## Project Setup

- `activate_project` — switch active project by name or path
- `get_current_config` — see active project, available projects, modes
- `onboarding` — call once per project to generate initial memory
- `initial_instructions` — read once per session if unsure how to drive Serena

## Limits and Fallbacks

- LSP must support the language. For dynamic/eval'd symbols, Blade templates, or config arrays, fall back to `grep`/`read`.
- `find_referencing_symbols` requires `relative_path` (a file, not a directory). Get it from a prior `find_symbol` result. For external deps, `relative_path` is an `<ext...>` identifier — never guess it.
- Line numbers are 0-based. Adjust when reporting or matching `read` output (1-based).
- Use `depth` before `include_body=true`. Request bodies only for specific method(s) needed.
- Batch related queries — call `find_symbol`, `find_references`, and `find_implementations` in parallel when independent.

## Anti-Patterns

- Don't `read` an entire file to find one method — use `find_symbol` with `include_body=true`.
- Don't `grep` for a class name or method usages — use `find_symbol` / `find_referencing_symbols`.
- Don't manually find-and-replace for a rename — use `rename_symbol`.
- Don't call `initial_instructions` more than once per session.

