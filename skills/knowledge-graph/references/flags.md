# Opt-in flags and secondary triggers

Load only when the flag was passed or the feature is invoked. Scripts live in `scripts.md` under the matching heading; tool semantics in `knowledge-graphs.md`.

## --obsidian

Skip unless given. `Obsidian vault` script — run before Step 9 while `graphify-out/.graphify_labels.json` is still available:

<!-- script: see references/scripts.md §Step 6 - Obsidian vault (only if --obsidian) -->

## --wiki

Agent-crawlable export — `index.md` plus one article per community and god-node articles. Run before Step 9 while `graphify-out/.graphify_labels.json` is still available:

<!-- script: see references/scripts.md §Step 6b - Wiki (only if --wiki) -->

## --neo4j / --neo4j-push <uri>

`--neo4j` — Cypher file for manual import:
<!-- script: see references/scripts.md §Step 7 - Neo4j export (only if --neo4j) -->

`--neo4j-push <uri>` — push to a running Neo4j; ask for credentials if not provided:
<!-- script: see references/scripts.md §Step 7 - Neo4j push (only if --neo4j-push) -->

Defaults: URI `bolt://localhost:7687`, user `neo4j`. MERGE-based — safe to re-run, no duplicates.

## --svg / --graphml

<!-- script: see references/scripts.md §Step 7b - SVG export (only if --svg) -->
<!-- script: see references/scripts.md §Step 7c - GraphML export (only if --graphml) -->

## --mcp

<!-- script: see references/scripts.md §Step 7d - MCP server (only if --mcp) -->

Stdio MCP server exposing `query_graph`, `get_node`, `get_neighbors`, `get_community`, `god_nodes`, `graph_stats`, `shortest_path`. Register in an MCP orchestrator (e.g. Claude Desktop `mcpServers.graphify`): command `python3`, args `["-m", "graphify.serve", "/absolute/path/to/graphify-out/graph.json"]`.

## --watch

Background watcher; auto-updates the graph on file changes:

<!-- script: see references/scripts.md §--watch -->

- **Code files only:** re-runs AST + rebuild + cluster immediately, no LLM. `graph.json`/`GRAPH_REPORT.md` update automatically.
- **Docs/papers/images:** writes `graphify-out/needs_update` flag; notify to run `/graphify --update` (LLM required).

Debounce (default 3s): waits for file activity to stop so parallel writes trigger one rebuild. Ctrl+C to stop.

## git commit hook

Post-commit hook auto-rebuilds the graph after every commit — no background process:

<!-- script: see references/scripts.md §git commit hook -->

Detects changed code files, re-runs AST on them, rebuilds `graph.json`/`GRAPH_REPORT.md`. Doc/image changes ignored — run `/graphify --update` manually.

## Devin always-on context

Run once per project:

<!-- script: see references/scripts.md §Devin always-on context -->

Writes always-on graphify context for Devin (skill under `~/.config/devin/skills/graphify/`) instructing the agent to check the graph before codebase questions and rebuild after code changes. `graphify devin uninstall` removes it.
