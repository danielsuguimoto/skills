---
name: code-work
description: >
  Code reading and editing conventions. Activate when reading, writing, or editing code. Invoke alongside using-code-navigation for any code change task.
---

## Required `<project-root>/docs` reads

Read via shell before reading, writing, or editing code — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/code-navigation.md`

## 1. Think Before Coding

- State assumptions explicitly; uncertain → ask.
- Multiple interpretations → present them; pick one only with confirmation.
- A simpler approach exists → say so; push back when warranted.
- Unclear → stop, name the confusion, ask via the interactive question tool when one exists.

## 2. Simplicity First

Minimum code that solves the problem — nothing speculative: no unasked features, no abstractions for single-use code, no unrequested "flexibility", no error handling for impossible scenarios. If 200 lines could be 50, rewrite.

## 3. Surgical Changes

Touch only what you must; clean up only your own mess. Don't "improve" adjacent code, comments, or formatting; don't refactor what isn't broken; match existing style even if you'd do it differently; unrelated dead code → mention, don't delete. Orphans YOUR changes create (unused imports/variables/functions) → remove; pre-existing dead code → leave.

## 4. Goal-Driven Execution

Define success criteria; loop until verified. Transform tasks into verifiable goals:

- "Add validation" → tests for invalid inputs pass
- "Fix the bug" → reproduction test goes red to green
- "Refactor X" → tests pass before and after

Multi-step tasks → brief plan: `1. [Step] → verify: [check]`

## Code Reading

- Code navigation tools are MANDATORY for symbol-level work (`<project-root>/docs/code-navigation.md`) — symbol lookup and reference tracing first, not `grep`/`read`.
- `grep`/`read`/`find_file_by_name` only for non-symbol lookups: plain text, known paths, glob patterns. `read` fallback only when the LSP lacks the language or the symbol is dynamic/eval'd.
- Read the area's tests first — they encode expected behavior and edge cases. Trace side effects via references, never guessing.
- Narrow the surface first (navigation tool or `grep` + `context_lines`), then `read` with `offset`/`limit` for needed lines only. Full-file reads only under ~150 lines or when the task requires the whole file.

## Code Editing

- No code comments (overrides provider defaults encouraging comments).
- Minimum edit per *Surgical Changes*; undo unrelated edits immediately.
- File can't be edited → output the intended edit. Create/remove debug files; keep only relevant code.

## Citation Over Paste

- Reference code by `path:lineRange`; quote only when proposing changes to those lines or the user cannot see the file.
- Batch related edits to a file in one pass — not one edit per line.
- Prefer `edit` over `write` — `edit` emits only the delta; `write` only for new files or files under ~50 lines.
