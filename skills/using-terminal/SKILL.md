---
name: using-terminal
description: MANDATORY HOW-skill. Must be invoked BEFORE sending ANY command to the terminal/shell tool. If this skill is not active in the current conversation, DO NOT call exec.
---

# RTK — Rust Token Killer

## Pre-flight checklist

Before every `exec` call, verify:
1. The command is prefixed with `rtk` (or uses a matching `rtk <subcommand>`).
2. No `cd` in the command. The shell already runs in the correct `cwd`. Use absolute paths for any file argument. `cd` is allowed only when the destination differs from `cwd` AND no absolute-path form exists. If unsure, omit `cd`.
3. If the command was suggested by another skill (e.g. `kool-cli`), that skill was also invoked.

Anti-`cd` examples — bad vs good:
- Bad: `cd /Users/jun/.agents && rtk ls` · Good: `rtk ls /Users/jun/.agents`
- Bad: `cd src && rtk grep foo` · Good: `rtk grep foo /abs/path/src`

## Rule

Prefix commands with `rtk` and use the matching RTK subcommand instead of the raw binary. If no subcommand exists, fall back to `rtk run <cmd>` (raw, unfiltered) or `rtk proxy <cmd>` (raw but tracked).

```bash
rtk git status
rtk cargo test
rtk npm run test
```

## Subcommand map (common)

Most raw commands map to `rtk <subcommand>`:

| Category | Examples |
| --- | --- |
| Filesystem | `rtk ls`, `rtk tree`, `rtk read <file>`, `rtk grep`, `rtk find`, `rtk wc`, `rtk diff` |
| Git | `rtk git …`, `rtk gh …`, `rtk glab …`, `rtk gt …` |
| Containers/Cloud | `rtk docker …`, `rtk kubectl …`, `rtk aws …`, `rtk psql …` |
| JS/TS | `rtk npm run …`, `rtk pnpm …`, `rtk npx …`, `rtk next …`, `rtk jest`, `rtk vitest`, `rtk playwright`, `rtk prisma …` |
| Rust/Go | `rtk cargo …`, `rtk go …`, `rtk golangci-lint run` |
| Python | `rtk pip …`, `rtk pytest`, `rtk ruff`, `rtk mypy` |
| Ruby | `rtk rspec`, `rtk rubocop`, `rtk rake` |
| JVM | `rtk mvn …`, `rtk gradlew …` |
| Lint (special) | `eslint` → `rtk lint`, `prettier --check` → `rtk prettier` |
| Network | `rtk curl`, `rtk wget`, `rtk env` |

For anything else: `rtk run <cmd>` (raw, unfiltered) or `rtk proxy <cmd>` (raw, tracked).

## Meta commands

```bash
rtk gain              # Show token savings
rtk gain --history    # Command history with savings
rtk discover          # Find missed RTK opportunities
rtk session           # RTK adoption across recent sessions
rtk rewrite <cmd>     # Print the RTK equivalent of a raw command
```

## Exceptions — do NOT use rtk

- Commands needing byte-exact stdout (piping binary, checksums, `rtk verify`).
- Interactive/TUI programs (`vim`, `top`, `tmux`, REPLs).
- Commands inside `rtk run`/`rtk proxy` are already raw — do not double-wrap.
- `kool …`: kool wraps Docker/docker-compose invocations and manages its own output formatting. Wrapping it in rtk breaks script resolution and output handling. Run `kool` commands directly.
- When a project rule or the user explicitly disables RTK for a command.

- Use `rtk <subcommand>` over the raw binary. RTK compresses output before it reaches context. It is the biggest token saver in terminal work.
- Pipe through `head`/`tail`/`grep`/`wc` for a slice of large output: `rtk logs app | tail -50`.
- Invoke `kool-cli` before any `kool` command.

## Output Truncation Awareness

Long output truncates to an overflow file you must re-read. That is pure waste. Prevent it up front:

- Pass scope flags before running: `--stat`, `--oneline`, `--name-only`, `--count`, `-q`.
- Pipe through `head`/`tail`/`grep`/`wc` for the decisive slice.
- `git log`/`diff`: always `--no-pager` + `--oneline` or `--stat`; never raw full diff.
- Test/lint suites: use compact mode (`--compact`, `--no-progress`, summary-only) over verbose per-test output.

- If output will likely exceed ~200 lines, narrow the command before running instead of re-reading the overflow.

