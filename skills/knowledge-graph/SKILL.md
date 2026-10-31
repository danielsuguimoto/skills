---
name: knowledge-graph
description: "Use for questions about a codebase, its architecture, or file relationships — or to turn inputs (code, docs, papers, images, videos) into a persistent knowledge graph with community detection and query/path/explain tools."
argument-hint: "[path|query|subcommand]"
---

## Required `<project-root>/docs` reads

Shell-read before any knowledge graph query or build; may be gitignored — missing → note gap, never invent: `<project-root>/docs/knowledge-graphs.md`

# /graphify — tool interface: `knowledge-graphs.md`

## Usage

| Command | Purpose |
|---------|---------|
| `/graphify [path]` | Full pipeline (current dir if no path) |
| `--mode deep` | Thorough extraction, richer INFERRED edges |
| `--directed` | Preserve edge direction (DiGraph) |
| `--whisper-model <name>` | Whisper model for video transcription (default `base`) |
| `--update` | Incremental: re-extract new/changed files only |
| `--cluster-only` | Rerun clustering on existing graph |
| `--no-viz` | Skip visualization, report + JSON only |
| `--obsidian` | Generate Obsidian vault (one file per node) |
| `--wiki` | Build agent-crawlable wiki |
| `--svg`/`--graphml` | Export graph.svg/graph.graphml |
| `--neo4j` / `--neo4j-push <url>` | Export Cypher file / push to running Neo4j |
| `--mcp` | Start MCP stdio server |
| `--watch` | Watch folder, auto-rebuild on changes |
| `add <url>` | Fetch URL, save to ./raw, update graph |
| `query "<q>"` / `path "A" "B"` / `explain "X"` | Query operations (`--dfs` chain tracing, `--budget N` token cap) |

`UPPER_SNAKE` tokens in scripts (`INPUT_PATH`, `LABELS_DICT`, `IS_DIRECTED`, `NEO4J_URI`…) are placeholders — substitute actual values from the invocation.

## When Invoked

`/graphify --help` or `-h` (no other args) → print `## Usage` verbatim and stop. No path → use `.` — do not ask.

### Step 1 - Ensure graphify is installed

<!-- script: see references/scripts.md §Step 1 - Ensure graphify is installed -->

Import succeeds → print nothing, go to Step 2.

### Step 2 - Detect files

<!-- script: see references/scripts.md §Step 2 - Detect files -->

Read silently — present a clean summary:

```
Corpus: X files · ~Y words
  code:     N files (.py .ts .go ...)
  docs:     N files (.md .txt ...)
  papers:   N files (.pdf ...)
  images:   N files
  video:    N files (.mp4 .mp3 ...)
```

Omit categories with 0 files. Then act:

- `total_files` 0 → stop: "No supported files found in [path]."
- `skipped_sensitive` non-empty → mention the count skipped, not the names.
- `total_words` > 2,000,000 OR `total_files` > 200 → show the warning + top 5 subdirectories by file count, ask which subfolder; wait before proceeding.
- Otherwise → Step 2.5 if video files detected, else Step 3.

### Step 2.5 - Transcribe video/audio (only if video files detected)

Skip if `detect` returned zero `video` files. Transcribe to text, treat as docs in Step 3.

**Step 1 - Whisper prompt:** write a one-sentence domain hint as `GRAPHIFY_WHISPER_PROMPT` (`knowledge-graphs.md` for Whisper config). Skim file names in `graphify-out/.graphify_detect.json`, compose from dominant topics (e.g. `transformer.py`, `attention.md` → `"Machine learning research on transformer architectures. Use proper punctuation."`). Video-only corpus fallback: `"Use proper punctuation and paragraph breaks."`

**Step 2 - Transcribe:** <!-- script: see references/scripts.md §Step 2.5 - Transcribe video / audio files -->

After: read `graphify-out/.graphify_transcripts.json`, add to the docs list for Part B, print `Transcribed N video file(s) -> treating as docs`. Warn on failures and continue.

**Whisper model:** default `base`; `--whisper-model <name>` → `GRAPHIFY_WHISPER_MODEL=<name>`.

### Step 3 - Extract entities and relationships

`--mode deep` given → `DEEP_MODE=true` on every extraction chunk in Step B2 — track from the original invocation.

Two parts: **structural** (AST, deterministic, free) and **semantic** (your model, costs tokens).

> **No external API key needed — never ask or block on one.** Code extracts structurally (AST); semantic extraction (docs/papers/images only) uses your model.

**Run Part A (AST) and Part B (semantic) in parallel — start both in the same message, merge in Part C.**

#### Part A - Structural extraction for code files

For code files detected, run AST extraction (`knowledge-graphs.md`):

<!-- script: see references/scripts.md §Step 3 Part A - Structural extraction (AST) -->

#### Part B - Semantic extraction (parallel chunks)

**Fast path:** zero docs/papers/images (code-only corpus) → skip Part B: write empty `.graphify_semantic.json` (`{"nodes":[],"edges":[],"hyperedges":[],"input_tokens":0,"output_tokens":0}`), go to Part C.

Print a timing estimate first: `ceil(uncached_non_code_files / 22)` chunks, ~45s per chunk batch → `"Semantic extraction: ~N files → X chunks, estimated ~Ys"`.

**B0 - Check extraction cache:**

<!-- script: see references/scripts.md §Step B0 - Check extraction cache -->

Process only files in `graphify-out/.graphify_uncached.txt`; all cached → skip to Part C.

**B1 - Split into chunks:** load `graphify-out/.graphify_uncached.txt`; chunks of 20-25 files; each image its own chunk; group same-directory files together.

**B2 - Extract each chunk — ALL in parallel.** For each chunk run the extraction prompt in `references/extraction-prompt.md`, filling `FILE_LIST`.

**B3 - Cache and merge.** Wait for all chunks. `graphify-out/.graphify_chunk_N.json` on disk = success signal. Valid JSON with `nodes`+`edges` → include and save to cache. Missing/invalid → warn, skip the chunk, do not abort. More than half failed/missing → stop and tell the user to re-run.

Update the chunk JSON's placeholder zero token counts with real values before merging:

<!-- script: see references/scripts.md §Step B3 - Merge chunks -->

Save new results to cache:
<!-- script: see references/scripts.md §Step B3 - Save new results to cache -->

Merge cached + new into `graphify-out/.graphify_semantic.json`:
<!-- script: see references/scripts.md §Step B3 - Merge cached + new results -->

Clean up: `rm -f graphify-out/.graphify_cached.json graphify-out/.graphify_uncached.txt graphify-out/.graphify_semantic_new.json`

#### Part C - Merge AST + semantic into final extraction

<!-- script: see references/scripts.md §Part C - Merge AST + semantic into final extraction -->

### Step 4 - Build graph, cluster, analyze, generate outputs

`build_from_json()` takes `directed=IS_DIRECTED` — `True` if `--directed` was given, else `False`.

<!-- script: see references/scripts.md §Step 4 - Build graph, cluster, analyze, generate outputs -->

`ERROR: Graph is empty` → stop and tell the user; do not proceed.

### Step 5 - Label communities

Read `graphify-out/.graphify_analysis.json`; name each community in 2-5 plain-language words (e.g. "Attention Mechanism", "Training Pipeline"). Regenerate the report and save labels (`LABELS_DICT` = e.g. `{0: "Attention Mechanism", 1: "Training Pipeline"}`):

<!-- script: see references/scripts.md §Step 5 - Label communities -->

### Step 6 - HTML visualization (always, unless --no-viz)

<!-- script: see references/scripts.md §Step 6 - HTML visualization -->

### Step 6b - Opt-in flags

`--obsidian` · `--wiki` · `--neo4j`/`--neo4j-push` · `--svg`/`--graphml` · `--mcp` → `references/flags.md` (load only when passed). Run before Step 9 while `graphify-out/.graphify_labels.json` exists.

### Step 8 - Token reduction benchmark (only if total_words > 5000)

<!-- script: see references/scripts.md §Step 8 - Token reduction benchmark (only if total_words > 5000) -->

Print output in chat. `total_words <= 5000` → skip silently.

### Step 9 - Save manifest, update cost tracker, clean up, report

<!-- script: see references/scripts.md §Step 9 - Save manifest, update cost tracker, clean up -->

Tell the user (omit `obsidian/` unless `--obsidian`, `wiki/` unless `--wiki`):
```
Graph complete. Outputs in PATH_TO_DIR/graphify-out/

  graph.html            - interactive graph, open in browser
  GRAPH_REPORT.md       - audit report
  graph.json            - raw graph data
  obsidian/             - Obsidian vault (only if --obsidian was given)
  wiki/                 - agent-crawlable wiki, start at wiki/index.md (only if --wiki was given)
```

Then paste only these GRAPH_REPORT.md sections — never the full report: **God Nodes** · **Surprising Connections** · **Suggested Questions**. Include the token cost (`This run`/`All time`); cohesion scores as raw numbers, not symbols.

Then offer to explore: pick the single most interesting suggested question (most community boundaries crossed / most surprising bridge node) and ask:

> "The most interesting question this graph can answer: **[question]**. Want me to trace it?"

Yes → run `/graphify query "[question]"` and walk the answer through the graph structure — which nodes connect, which community boundaries cross, what the path reveals. Keep going as long as they want; end each answer with a natural follow-up ("this connects to X — want to go deeper?").

---

## Interpreter guard for subcommands

Before any subcommand (`--update`, `--cluster-only`, `query`, `path`, `explain`, `add`), check `.graphify_python` exists; missing → re-resolve the interpreter first:

<!-- script: see references/scripts.md §Interpreter guard for subcommands -->

---

## For --update (incremental re-extraction)

Use after files were added/modified since the last run; only re-extracts changed files.

<!-- script: see references/scripts.md §--update - Detect incremental changes -->

New files exist → check whether all changed files are code:

<!-- script: see references/scripts.md §--update - Check if code-only -->

`code_only` True → print `[graphify update] Code-only changes detected - skipping semantic extraction (no LLM needed)`, run Part A only, skip Part B, then Part C + Steps 4-8. False → full Step 3 pipeline (Parts A-C).

No new files (only deletions) → create an empty extraction so merge can prune:

<!-- script: see references/scripts.md §--update - Empty extraction for deletions only -->

Save the old graph for the diff: `cp graphify-out/graph.json graphify-out/.graphify_old.json`

<!-- script: see references/scripts.md §--update - Merge new extraction into existing graph -->

Run Steps 4-8 on the merged graph. After Step 4, show the graph diff:

<!-- script: see references/scripts.md §--update - Graph diff -->

Clean up: `rm -f graphify-out/.graphify_old.json`

---

## For --cluster-only

Skip Steps 1-3. Load `graphify-out/graph.json`, re-run clustering:

<!-- script: see references/scripts.md §--cluster-only -->

Then Steps 5-9 as normal.

---

## For /graphify query, path, and explain

See `queries.md` for detailed examples.

---

## For /graphify add

Fetch a URL, add to the corpus, update the graph:

<!-- script: see references/scripts.md §/graphify add - Fetch URL and add to corpus -->

Set `AUTHOR`/`CONTRIBUTOR` to the user's name when provided. After a successful save, run `--update` on `./raw` to merge the file.

URL types (auto-detected): Twitter/X → oEmbed → `.md` · arXiv → abstract+metadata `.md` · PDF → `.pdf` · images → downloaded, vision extraction next build · any webpage → html2text → markdown.

---

## For --watch · git commit hook · Devin always-on context

`references/flags.md` — load for these triggers.
