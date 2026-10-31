---
name: container-cli
description: "Container CLI usage reference. Invoke when the project uses a container CLI (config file present) and a container command is needed — start/stop/restart, exec, logs, run scripts, share tunnels."
---

## Required `<project-root>/docs` reads

Shell-read before running any container CLI command; may be gitignored — missing → note gap, never invent: `<project-root>/docs/container-clis.md`

## Before Running Any Container CLI Command

- Confirm the tool's config file exists (the spec names it). If missing, stop and ask — the project may not use this container CLI.
- Read the config file once for service names and script names — never guess or invent them. Recall what it showed; re-read only when it changes.
- Use the tool's list/status operations to enumerate scripts and running services when unsure.

## Important Rules

- Always run from the project root (the directory holding the config file) or use the tool's working-directory flag.
- Purge/`-v`/destroy flags are destructive — they delete persistent volume data.
- Prefer JSON/machine-readable output when parsing; report only the error or signal from logs (`| tail -N`, `| grep`).
