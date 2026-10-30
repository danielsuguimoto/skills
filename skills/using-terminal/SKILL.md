---
name: using-terminal
description: MANDATORY skill for safe and efficient terminal/shell command execution.
---

## Required `<project-root>/docs` reads

Read via shell before running any terminal/shell command — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/container-clis.md`
- `<project-root>/docs/terminal-wrappers.md`
- `<project-root>/docs/terminal-tools.md`

# Terminal Token Compression

## Pre-flight checklist

Before every terminal/shell command, verify:

1. Terminal wrapper configured in `<project-root>/docs/terminal-wrappers.md`?
2. The command uses the configured terminal wrapper (or a matching wrapper subcommand).
3. No `cd` in the command. The shell runs in the correct `cwd`. Use absolute paths for file arguments. Use `cd` only when the destination differs from `cwd` and no absolute-path form exists. If unsure, omit `cd`.

Anti-`cd` examples:

- Bad: `cd /Users/jun/.agents && terminal-wrapper ls` · Good: `terminal-wrapper ls /Users/jun/.agents` (e.g. `rtk ls`)
- Bad: `cd src && terminal-wrapper grep foo` · Good: `terminal-wrapper grep foo /abs/path/src` (e.g. `rtk grep`)

## Rule

If a terminal wrapper is configured in `<project-root>/docs/terminal-wrappers.md`, prefix commands with it and use the matching wrapper subcommand instead of the raw binary. If no subcommand exists, fall back to the wrapper's raw run or raw proxy command.

If no wrapper is configured or `<project-root>/docs/terminal-wrappers.md` is missing/empty, run raw commands without any wrapper.

```bash
terminal-wrapper git status
terminal-wrapper cargo test
terminal-wrapper npm run test
```

## Subcommand map

The wrapper maps raw commands to compressed subcommands — full map in `<project-root>/docs/terminal-wrappers.md`.

## Meta commands

```bash
terminal-wrapper gain              # Show token savings
terminal-wrapper gain --history    # Command history with savings
terminal-wrapper discover          # Find missed terminal wrapper opportunities
terminal-wrapper session           # Terminal wrapper adoption across recent sessions
terminal-wrapper rewrite <cmd>     # Print the terminal wrapper equivalent of a raw command
```

## Exceptions — do NOT use the terminal wrapper

- Commands needing byte-exact stdout (piping binary, checksums, terminal wrapper verify).
- Interactive/TUI programs (`vim`, `top`, `tmux`, REPLs).
- Commands inside the wrapper's raw run/proxy are already raw — do not double-wrap.
- Container CLI (`<project-root>/docs/container-clis.md`): invoke the container CLI skill before any container CLI command. The container CLI wraps Docker/docker-compose invocations and manages its own output formatting — wrapping it breaks script resolution and output handling. Run container CLI commands directly.
- A project rule or the user explicitly disables the terminal wrapper for a command.

## Output Truncation Awareness

Large output overflows into a file you must re-read. Avoid it:

- Pass scope flags first: `--stat`, `--oneline`, `--name-only`, `--count`, `-q`.
- Pipe through `head`/`tail`/`grep`/`wc` for the decisive slice.
- `git log`/`diff`: always `--no-pager` + `--oneline` or `--stat`; never raw full diff.
- Test/lint suites: use compact mode (`--compact`, `--no-progress`, summary-only) over verbose per-test output.
- If output will likely exceed ~200 lines, narrow the command before running instead of re-reading the overflow.
