# Graph build and update

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

For `--update`, `--cluster-only`, or `add`, follow the matching section below. Otherwise run the build steps; no path → use `.`.

### Step 1 - Ensure graphify is installed

<!-- script: see references/scripts.md §Step 1 - Ensure graphify is installed -->

Import succeeds → print nothing, go to Step 2.

### Step 2 - Detect files

<!-- script: see references/scripts.md §Step 2 - Detect files -->

Read the detection result, then act:

- `total_files` 0 → stop: "No supported files found in [path]."
- `skipped_sensitive` non-empty → mention the count skipped, not the names.
- `total_words` > 2,000,000 OR `total_files` > 200 → show the warning + top 5 subdirectories by file count, ask which subfolder; wait before proceeding.
- Otherwise → Step 2.5 if video files detected, else Step 3.

### Step 2.5 - Transcribe video/audio (only if video files detected)

Skip if `detect` returned zero `video` files. Transcribe to text, treat as docs in Step 3.

**Step 1 - Whisper prompt:** write a one-sentence domain hint as `GRAPHIFY_WHISPER_PROMPT` (`knowledge-graphs.md` for Whisper config). Skim file names in `graphify-out/.graphify_detect.json`, compose from dominant topics (e.g. `transformer.py`, `attention.md` → `"Machine learning research on transformer architectures. Use proper punctuation."`). Video-only corpus fallback: `"Use proper punctuation and paragraph breaks."`

**Step 2 - Transcribe:** <!-- script: see references/scripts.md §Step 2.5 - Transcribe video / audio files -->

After: read `graphify-out/.graphify_transcripts.json` and add transcripts to the docs list for Part B. Warn on failures and continue.

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

### Step 8 - Token reduction benchmark (only when requested and total_words > 5000)

<!-- script: see references/scripts.md §Step 8 - Token reduction benchmark (only if total_words > 5000) -->

Report the result when run; otherwise skip.

### Step 9 - Save manifest, update cost tracker, clean up, report

<!-- script: see references/scripts.md §Step 9 - Save manifest, update cost tracker, clean up -->

Report the output directory, requested outputs, and any failures. Summarize report findings only when requested; do not offer unsolicited follow-up queries.

---

## For --update (incremental re-extraction)

Use after files were added/modified since the last run; only re-extracts changed files.

<!-- script: see references/scripts.md §--update - Detect incremental changes -->

New files exist → check whether all changed files are code:

<!-- script: see references/scripts.md §--update - Check if code-only -->

`code_only` True → run Part A only, skip Part B, then Part C + Steps 4-8. False → full Step 3 pipeline (Parts A-C).

No new files (only deletions) → create an empty extraction so merge can prune:

<!-- script: see references/scripts.md §--update - Empty extraction for deletions only -->

Save the old graph for the diff: `cp graphify-out/graph.json graphify-out/.graphify_old.json`

<!-- script: see references/scripts.md §--update - Merge new extraction into existing graph -->

Run Steps 4-8 on the merged graph. After Step 4, compute the graph diff:

<!-- script: see references/scripts.md §--update - Graph diff -->

Clean up: `rm -f graphify-out/.graphify_old.json`

---

## For --cluster-only

Skip Steps 1-3. Load `graphify-out/graph.json`, re-run clustering:

<!-- script: see references/scripts.md §--cluster-only -->

Then Steps 5-9 as normal.

---

## For /graphify add

Fetch a URL, add to the corpus, update the graph:

<!-- script: see references/scripts.md §/graphify add - Fetch URL and add to corpus -->

Set `AUTHOR`/`CONTRIBUTOR` to the user's name when provided. After a successful save, run `--update` on `./raw` to merge the file.

URL types (auto-detected): Twitter/X → oEmbed → `.md` · arXiv → abstract+metadata `.md` · PDF → `.pdf` · images → downloaded, vision extraction next build · any webpage → html2text → markdown.
