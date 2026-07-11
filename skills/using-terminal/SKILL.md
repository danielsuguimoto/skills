---
name: using-terminal
description: MANDATORY HOW-skill. Must be invoked BEFORE sending ANY command to the terminal/shell tool. If this skill is not active in the current conversation, DO NOT call exec.
---

# Terminal Token Compression

## Pre-flight checklist

Before every `exec` call, verify:
1. The command is prefixed with the terminal wrapper (see `/docs/terminal-wrappers.md` in the project root) (or uses a matching terminal wrapper subcommand).
2. No `cd` in the command. The shell already runs in the correct `cwd`. Use absolute paths for any file argument. `cd` is allowed only when the destination differs from `cwd` AND no absolute-path form exists. If unsure, omit `cd`.
3. If the command was suggested by another skill (e.g. the container CLI skill), that skill was also invoked.

Anti-`cd` examples — bad vs good:
- Bad: `cd /Users/jun/.agents && terminal-wrapper ls` · Good: `terminal-wrapper ls /Users/jun/.agents` (e.g., `rtk ls`)
- Bad: `cd src && terminal-wrapper grep foo` · Good: `terminal-wrapper grep foo /abs/path/src` (e.g., `rtk grep`)

## Rule

Prefix commands with the terminal wrapper (see `/docs/terminal-wrappers.md` in the project root) and use the matching terminal wrapper subcommand instead of the raw binary. If no subcommand exists, fall back to the terminal wrapper's raw run command (raw, unfiltered) or raw proxy command (raw but tracked).

```bash
terminal-wrapper git status
terminal-wrapper cargo test
terminal-wrapper npm run test
```

## Subcommand map (common)

The terminal wrapper maps raw commands to compressed subcommands (see `/docs/terminal-wrappers.md` in the project root for the full map). Common categories:

| Category | Examples |
| --- | --- |
| Filesystem | `terminal-wrapper ls`, `terminal-wrapper tree`, `terminal-wrapper read <file>`, `terminal-wrapper grep`, `terminal-wrapper find`, `terminal-wrapper wc`, `terminal-wrapper diff` |
| Git | `terminal-wrapper git …`, `terminal-wrapper gh …`, `terminal-wrapper glab …`, `terminal-wrapper gt …` |
| Containers/Cloud | `terminal-wrapper docker …`, `terminal-wrapper kubectl …`, `terminal-wrapper aws …`, `terminal-wrapper psql …` |
| JS/TS | `terminal-wrapper npm run …`, `terminal-wrapper pnpm …`, `terminal-wrapper npx …`, `terminal-wrapper next …`, `terminal-wrapper jest`, `terminal-wrapper vitest`, `terminal-wrapper playwright`, `terminal-wrapper prisma …` |
| Rust/Go | `terminal-wrapper cargo …`, `terminal-wrapper go …`, `terminal-wrapper golangci-lint run` |
| Python | `terminal-wrapper pip …`, `terminal-wrapper pytest`, `terminal-wrapper ruff`, `terminal-wrapper mypy` |
| Ruby | `terminal-wrapper rspec`, `terminal-wrapper rubocop`, `terminal-wrapper rake` |
| JVM | `terminal-wrapper mvn …`, `terminal-wrapper gradlew …` |
| Lint (special) | `eslint` → `terminal-wrapper lint`, `prettier --check` → `terminal-wrapper prettier` |
| Network | `terminal-wrapper curl`, `terminal-wrapper wget`, `terminal-wrapper env` |

For anything else: the terminal wrapper's raw run command (raw, unfiltered) or raw proxy command (raw, tracked) (see `/docs/terminal-wrappers.md` in the project root).

## Meta commands

The terminal wrapper exposes meta commands (see `/docs/terminal-wrappers.md` in the project root):

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
- Commands inside the terminal wrapper's raw run/proxy are already raw — do not double-wrap.
- Container CLI (see `/docs/container-clis.md` in the project root): the container CLI wraps Docker/docker-compose invocations and manages its own output formatting. Wrapping it in the terminal wrapper breaks script resolution and output handling. Run container CLI commands directly.
- When a project rule or the user explicitly disables the terminal wrapper for a command.

- Use the terminal wrapper subcommand over the raw binary (see `/docs/terminal-wrappers.md` in the project root). The terminal wrapper compresses output before it reaches context. It is the biggest token saver in terminal work.
- Pipe through `head`/`tail`/`grep`/`wc` for a slice of large output: `terminal-wrapper logs app | tail -50`.
- Invoke the container CLI skill before any container CLI command.

## Output Truncation Awareness

Long output truncates to an overflow file you must re-read. That is pure waste. Prevent it up front:

- Pass scope flags before running: `--stat`, `--oneline`, `--name-only`, `--count`, `-q`.
- Pipe through `head`/`tail`/`grep`/`wc` for the decisive slice.
- `git log`/`diff`: always `--no-pager` + `--oneline` or `--stat`; never raw full diff.
- Test/lint suites: use compact mode (`--compact`, `--no-progress`, summary-only) over verbose per-test output.

- If output will likely exceed ~200 lines, narrow the command before running instead of re-reading the overflow.

