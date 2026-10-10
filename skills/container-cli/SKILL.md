---
name: container-cli
description: "Container CLI usage reference. Invoke when the project uses a container CLI (config file present) and a container command is needed — start/stop/restart, exec, logs, run scripts, share tunnels."
---

## Required `<project-root>/docs` reads

Read these project-root spec files before running any container CLI command (use shell `cat`/`ls` — they may be in `.gitignore`, invisible to built-in search). Missing file → fall back to native tools, note the gap; never invent contents.

- `<project-root>/docs/container-clis.md`

Container CLIs wrap container runtimes. They manage container lifecycle, run commands inside services, execute project scripts, and sometimes share local environments via HTTP tunnels.

## Command Summary

The spec names the tool and documents its exact commands (see `<project-root>/docs/container-clis.md` in the project root). Typical operation areas:

| Area | Operations |
| --- | --- |
| Lifecycle | start, stop, restart, status |
| Exec in container | run a command inside a named service |
| One-off containers | ephemeral run against an image or service |
| Logs | tail/stream service logs |
| Scripts | named commands defined in the tool's config file |
| Share tunnel | expose a local service via HTTP tunnel (not all tools) |

## Before Running Any Container CLI Command

- Confirm the tool's config file exists (the spec names it). If missing, stop and ask — the project may not use this container CLI.
- Read the config file first to discover service names and script names. Never guess; verify they exist.
- Use the tool's list/status operations to enumerate scripts and running services when unsure.

## Important Rules

- Always run from the project root (the directory holding the config file) or use the tool's working-directory flag.
- Service names come from the config/compose file — never invent them.
- Script names come from the tool's config file. Confirm by reading the file or the tool's list operation.
- Purge/`-v`/destroy flags are destructive — they delete persistent volume data.
- Prefer JSON output when parsing (a `--json` flag or machine-readable mode).
- Filter logs with `| tail -N` or `| grep` and report only the error or signal.
- Read the config file once. Recall the scripts already discovered; re-read only when it changes.
