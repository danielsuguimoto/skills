---
name: kool-cli
description: "kool (kool.dev) CLI usage reference. Invoke when the project uses kool (kool.yml present) and any kool command is needed — start/stop/restart, exec, logs, run scripts, share tunnels."
---

kool wraps `docker-compose`. It manages container lifecycle, runs commands inside services, executes project scripts from `kool.yml`, and shares local environments via HTTP tunnels.

- Repo: https://github.com/kool-dev/kool
- Docs: https://kool.dev/docs | Command reference: https://kool.dev/docs/commands-reference

## Command Summary

| Area | Commands |
| --- | --- |
| Lifecycle | `kool start`, `kool stop [--purge]`, `kool restart [--rebuild]`, `kool status` |
| Exec in container | `kool exec <service> <cmd>`, `-e VAR=val`, `-d` (detached) |
| One-off containers | `kool docker [OPTS] IMAGE [CMD]` — opts: `-e`, `-n`, `-p`, `-v` |
| Logs | `kool logs [-f] [-t N]` (`-t 0` = all lines) |
| Scripts (kool.yml) | `kool run`, `kool run <script>`, `kool run <script> -- <args>` |
| Share tunnel | `kool share [--service S] [--port P] [--subdomain X]` |
| Setup | `kool create PRESET FOLDER`, `kool preset`, `kool recipe`, `kool info`, `kool self-update` |
| Global | `-w <path>`, `--verbose` |

Presets: Laravel, Laravel+Octane, Symfony, CodeIgniter, AdonisJs, NestJS, NextJS, NuxtJS, NodeJS, ExpressJS, Hugo, WordPress, PHP, Monorepo NestJS+NextJS.

## Before Running Any kool Command

- Read `kool.yml` first to discover scripts, and `docker-compose.yml` to discover service names. Never guess; verify they exist.
- Use `kool run` (or `--json`) to list scripts and `kool status` to list running services when unsure.
- If `kool.yml` or `docker-compose.yml` is missing, stop and ask — the project may not use kool.

## Important Rules

- Always run from the project root (the one with `docker-compose.yml` and `kool.yml`) or use `-w`.
- Service names come from `docker-compose.yml`.
- Script names come from `kool.yml`. Never invent them; confirm by reading the file or running `kool run`.
- Script args only work with single-line scripts. Multi-line scripts reject extra args (`ErrExtraArguments`).
- Scripts in `kool.yml` aren't full bash. Use `kool docker <image> bash -c "..."` for pipes and conditionals.
- `--purge` is destructive: removes all persistent data from volumes.
- `--json` on `kool run` only works without a script argument (lists scripts, doesn't run one).

- Use `--json` when parsing output (e.g., `kool run --json` to list scripts). JSON is more compact and parseable than human-readable tables.
- Read `kool.yml` once. Don't re-read it before every `kool run`; recall the scripts already discovered.
- Don't paste full `kool logs` output. Filter with `| tail -N` or `| grep` and report only the error or signal.

