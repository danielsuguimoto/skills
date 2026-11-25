---
name: container-cli
description: "Use a configured container CLI for lifecycle, exec, logs, scripts, or tunnels."
---

## Required `<project-root>/docs` reads

Shell-read before running any container CLI command (reuse session reads): `<project-root>/docs/container-clis.md`. If a spec is missing, report the gap and use a documented fallback.

## Before Running a Container CLI Command

- When the command requires project config, confirm the file named in the spec exists. If missing, stop and ask.
- For project-scoped commands, read the config once for service and script names; re-read only when it changes. Never guess names.
- Use the tool's list/status operations to enumerate scripts and running services when unsure.
- Config-free commands, such as logs for a named container, need no project config.

## Important Rules

- Run project-scoped commands from the project root (the directory holding the config file) or use the tool's working-directory flag.
- Purge/`-v`/destroy flags are destructive — they delete persistent volume data.
- Prefer JSON/machine-readable output when parsing; report only the error or signal from logs (`| tail -N`, `| grep`).
