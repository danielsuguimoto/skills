---
name: using-terminal
description: "Required before shell commands; apply wrapper and output limits."
---

## Required `<project-root>/docs` reads

Shell-read before running terminal/shell commands (reuse session reads): `<project-root>/docs/terminal-wrappers.md`, `<project-root>/docs/terminal-tools.md`, `<project-root>/docs/container-clis.md` (only before a container command). If a spec is missing, report the gap and use a documented fallback.

## Wrapper

Follow `terminal-wrappers.md` for which commands to wrap. Use the matching subcommand or the raw proxy. Without a configured wrapper, run raw commands. Run raw for byte-exact output, interactive tools, commands already inside a proxy, and project or user exceptions. For container commands, follow `container-clis.md` and run the container CLI directly.

Use absolute paths from the project's working directory. Change directories only when a command has no absolute-path form.

## Output

Scope output to the evidence you need before running. Use summary or quiet modes, file filters, or bounded slices. Disable the pager for Git; start diffs and logs with stats or one-line summaries. Read full hunks only when needed. Narrow output likely to exceed 200 lines.
