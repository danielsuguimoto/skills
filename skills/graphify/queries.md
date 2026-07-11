# graphify Query Examples

Examples for `query`, `path`, and `explain`. Referenced by `SKILL.md`.

## For /graphify query

Pick a traversal mode based on the question:

| Mode | Flag | Best for |
|------|------|----------|
| BFS (default) | _(none)_ | "What is X connected to?" - broad context, nearest neighbors first |
| DFS | `--dfs` | "How does X reach Y?" - trace a specific chain or dependency path |

First check the graph exists:
```bash
$(cat graphify-out/.graphify_python) -c "
from pathlib import Path
if not Path('graphify-out/graph.json').exists():
    print('ERROR: No graph found. Run /graphify <path> first to build the graph.')
    raise SystemExit(1)
"
```
If it fails, tell the user to run `/graphify <path>` first.

Load `graphify-out/graph.json`, then:

1. Find 1-3 nodes whose labels best match key terms in the question.
2. Run the right traversal from each starting node.
3. Read the subgraph: node labels, edge relations, confidence tags, source locations.
4. Answer using **only** what the graph contains. Quote `source_location` when citing a specific fact.
5. If the graph lacks enough information, say so — do not hallucinate edges.

<!-- script: see references/scripts.md §query-traversal -->

Replace `QUESTION`, `MODE` (`bfs` or `dfs`), and `BUDGET` (default `2000`, or `--budget N`). Then answer from the subgraph.

After writing the answer, save it back to improve future queries:

```bash
$(cat graphify-out/.graphify_python) -m graphify save-result --question "QUESTION" --answer "ANSWER" --type query --nodes NODE1 NODE2
```

---

## For /graphify path

Find the shortest path between two named concepts.

First check the graph exists:
```bash
$(cat graphify-out/.graphify_python) -c "
from pathlib import Path
if not Path('graphify-out/graph.json').exists():
    print('ERROR: No graph found. Run /graphify <path> first to build the graph.')
    raise SystemExit(1)
"
```

<!-- script: see references/scripts.md §query-path -->

Replace `NODE_A` and `NODE_B` with the actual concept names. Then explain the path in plain language: what each hop means and why it matters.

After writing the explanation, save it back:

```bash
$(cat graphify-out/.graphify_python) -m graphify save-result --question "Path from NODE_A to NODE_B" --answer "ANSWER" --type path_query --nodes NODE_A NODE_B
```

---

## For /graphify explain

Explain a single node and everything connected to it.

First check the graph exists:
```bash
$(cat graphify-out/.graphify_python) -c "
from pathlib import Path
if not Path('graphify-out/graph.json').exists():
    print('ERROR: No graph found. Run /graphify <path> first to build the graph.')
    raise SystemExit(1)
"
```

<!-- script: see references/scripts.md §query-explain -->

Replace `NODE_NAME` with the concept. Then write a 3-5 sentence explanation using source locations as citations.

After writing the explanation, save it back:

```bash
$(cat graphify-out/.graphify_python) -m graphify save-result --question "Explain NODE_NAME" --answer "ANSWER" --type explain --nodes NODE_NAME
```
