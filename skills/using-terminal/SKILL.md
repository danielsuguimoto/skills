---
name: using-terminal
description: MANDATORY skill for safe and efficient terminal/shell command execution.
---

## Required `<project-root>/docs` reads

Read via shell before running terminal/shell commands — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/container-clis.md`
- `<project-root>/docs/terminal-wrappers.md`
- `<project-root>/docs/terminal-tools.md`

# Terminal Token Compression

## Rule

A terminal wrapper configured in `<project-root>/docs/terminal-wrappers.md` prefixes every command — use the matching wrapper subcommand over the raw binary (full map in that file). No subcommand → the wrapper's raw run/proxy command. No wrapper configured, or the spec missing/empty → run raw commands.

```bash
terminal-wrapper git status
terminal-wrapper cargo test
terminal-wrapper npm run test
```

- No `cd` — the shell runs in the correct `cwd`; pass absolute paths (`rtk ls /Users/jun/.agents`, `rtk grep foo /abs/path/src`). `cd` only when no absolute-path form exists; if unsure, omit `cd`.
- Meta: `terminal-wrapper gain` (savings, `--history` for per-command) · `discover` (missed opportunities) · `session` (adoption) · `rewrite <cmd>`.

## Exceptions — do NOT wrap

- Byte-exact stdout (binary piping, checksums, wrapper verify).
- Interactive/TUI programs (`vim`, `top`, `tmux`, REPLs).
- Commands already inside the wrapper's raw run/proxy — no double-wrap.
- Container CLI (`<project-root>/docs/container-clis.md`): invoke the container-cli skill before container commands; it manages its own output formatting — run container CLI commands directly.
- A project rule or the user disables the wrapper for a command.

## Output Truncation Awareness

Large output overflows into a file you must re-read — narrow first:

- Scope flags first: `--stat`, `--oneline`, `--name-only`, `--count`, `-q`.
- Pipe `head`/`tail`/`grep`/`wc` for the decisive slice.
- `git log`/`diff`: always `--no-pager` + `--oneline`/`--stat`; never raw full diff.
- Test/lint suites: compact mode (`--compact`, `--no-progress`, summary-only) over per-test output.
- Expecting >~200 lines → narrow the command instead of re-reading the overflow.
