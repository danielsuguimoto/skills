# graphify Query Examples

`query`, `path`, `explain` are native `graphify` subcommands — invoked as `$(cat graphify-out/.graphify_python) -m graphify <cmd>`. All take `--graph <path>` (default `graphify-out/graph.json`). Missing graph → tell the user to run `/graphify <path>` first.

## For /graphify query

Pick a traversal mode:

| Mode | Flag | Best for |
|------|------|----------|
| BFS (default) | — | "What is X connected to?" — broad context, nearest neighbors first |
| DFS | `--dfs` | "How does X reach Y?" — trace a chain or dependency path |

```bash
$(cat graphify-out/.graphify_python) -m graphify query "QUESTION" [--dfs] [--budget N]
```

`--budget N` caps output tokens (default 2000). Answer using **only** what the graph contains — quote `source_location` when citing a fact; insufficient info → say so, never invent edges.

Save the result back to improve future queries:

```bash
$(cat graphify-out/.graphify_python) -m graphify save-result --question "QUESTION" --answer "ANSWER" --type query --nodes NODE1 NODE2
```

## For /graphify path

```bash
$(cat graphify-out/.graphify_python) -m graphify path "NODE_A" "NODE_B"
```

Explain the path in plain language: what each hop means and why it matters. Save with `--type path_query`.

## For /graphify explain

```bash
$(cat graphify-out/.graphify_python) -m graphify explain "NODE_NAME"
```

Write a 3-5 sentence explanation using source locations as citations. Save with `--type explain`.
