---
name: using-terminal
description: "MANDATORY — terminal/shell command execution: wrapper usage and output truncation rules."
---

## Required `<project-root>/docs` reads

Shell-read before running terminal/shell commands; may be gitignored — missing → note gap, never invent: `<project-root>/docs/container-clis.md`, `<project-root>/docs/terminal-wrappers.md`, `<project-root>/docs/terminal-tools.md`

# Terminal Token Compression

A terminal wrapper configured in `terminal-wrappers.md` prefixes every command — use the matching wrapper subcommand over the raw binary (full map in that file). No subcommand → the wrapper's raw run/proxy command. No wrapper configured, or the spec missing/empty → run raw commands.

```bash
terminal-wrapper <cmd>     # e.g. rtk git status · rtk cargo test
```

- No `cd` — the shell runs in the correct `cwd`; pass absolute paths (`rtk ls /abs/path`, `rtk grep foo /abs/path/src`). `cd` only when no absolute-path form exists; if unsure, omit `cd`.
- Meta: `terminal-wrapper gain` (savings, `--history` for per-command) · `discover` (missed opportunities) · `session` (adoption) · `rewrite <cmd>`.

## Exceptions — do NOT wrap

- Byte-exact stdout (binary piping, checksums, wrapper verify).
- Interactive/TUI programs (`vim`, `top`, `tmux`, REPLs).
- Commands already inside the wrapper's raw run/proxy — no double-wrap.
- Container CLI (`container-clis.md`): invoke the container-cli skill before container commands; it manages its own output formatting — run container CLI commands directly.
- A project rule or the user disables the wrapper for a command.

## Output Truncation Awareness

Large output overflows into a file you must re-read — narrow first:

- Scope flags (`--stat`, `--oneline`, `--name-only`, `--count`, `-q`) or `head`/`tail`/`grep`/`wc` for the decisive slice.
- `git log`/`diff`: `--no-pager` + `--oneline`/`--stat`; never raw full diff.
- Test/lint suites: compact mode (`--compact`, `--no-progress`, summary-only) over per-test output.
- Expecting >~200 lines → narrow the command instead of re-reading the overflow.
