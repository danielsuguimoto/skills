---
name: container-cli
description: "Use a configured container CLI for lifecycle, exec, logs, scripts, or tunnels."
---

## Required `<project-root>/docs` reads

Shell-read before running any container CLI command (reuse session reads): `<project-root>/docs/container-clis.md`. If a spec is missing, report the gap and use a documented fallback.

## Before Running Any Container CLI Command

- Confirm the tool's config file exists (the spec names it). If missing, stop and ask — the project may not use this container CLI.
- Read the config file once for service names and script names — never guess or invent them. Recall what it showed; re-read only when it changes.
- Use the tool's list/status operations to enumerate scripts and running services when unsure.

## Important Rules

- Always run from the project root (the directory holding the config file) or use the tool's working-directory flag.
- Purge/`-v`/destroy flags are destructive — they delete persistent volume data.
- Prefer JSON/machine-readable output when parsing; report only the error or signal from logs (`| tail -N`, `| grep`).
