---
name: knowledge-graph
description: "Build, update, cluster, or query/path/explain a persistent knowledge graph from code, documents, papers, images, or videos; generate a wiki/report or answer architecture questions from an existing graph."
argument-hint: "[path|query|subcommand]"
---

Shell-read `<project-root>/docs/knowledge-graphs.md` before a graph operation. If missing, note the gap; do not invent tool interfaces.

## Route

- `--help` or `-h` alone: read the Usage table in [build.md](build.md), print it, and stop.
- `query`, `path`, or `explain`: apply the interpreter guard, then follow [queries.md](queries.md).
- Architecture question with an existing graph: apply the interpreter guard, then query it via [queries.md](queries.md). Do not build a graph unless requested.
- `--update`, `--cluster-only`, or `add`: apply the interpreter guard, then follow the matching section in [build.md](build.md).
- `--watch`, git commit hook, or always-on context: follow [flags.md](references/flags.md).
- Otherwise, follow the build steps in [build.md](build.md). Use `.` when no path is given.

Before a guarded subcommand, check `graphify-out/.graphify_python`; if missing, re-resolve the interpreter using the interpreter guard in [scripts.md](references/scripts.md).
