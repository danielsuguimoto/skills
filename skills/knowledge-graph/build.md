# Graph build and update

## Usage

| Command | Purpose |
|---------|---------|
| `/graphify [path]` | Full pipeline (current dir if no path) |
| `--mode deep` | Thorough extraction, richer INFERRED edges |
| `--directed` | Preserve edge direction (DiGraph) |
| `--whisper-model <name>` | Whisper model for video transcription (default `base`) |
| `--update` | Repair on changed/deleted input: fresh full extraction, semantic cache reused |
| `--cluster-only` | Rerun clustering on existing graph |
| `--no-viz` | Skip visualization, report + JSON only |
| `--obsidian` | Generate Obsidian vault (one file per node) |
| `--wiki` | Build agent-crawlable wiki |
| `--svg`/`--graphml` | Export graph.svg/graph.graphml |
| `--neo4j` / `--neo4j-push [<url>]` | Export Cypher file / push to running Neo4j — omitted URL reuses the session per the rule below |
| `--mcp` | Start MCP stdio server |
| `--watch` | Watch folder, auto-rebuild on changes |
| `add [<url>]` | Fetch URL, save under the corpus root's `raw/`, update graph — omitted URL reuses the session per the rule below |
| `query "<q>"` / `path "A" "B"` / `explain "X"` | Query operations (`--dfs` chain tracing, `--budget N` token cap) |

`UPPER_SNAKE` tokens in scripts (`INPUT_PATH`, `LABELS_DICT`, `IS_DIRECTED`, `NEO4J_URI`…) are placeholders — substitute resolved values. For `add`, use the explicit URL first, else the unambiguous source URL from the session's current work; missing or ambiguous → ask. For `--neo4j-push`, use the explicit URL first, else the unambiguous destination URL from the session's current work. If multiple destinations fit, ask before pushing; if none fit, use the documented default in `references/flags.md`. A session URL identifies the target only; it does not authorize an unrequested add or push.

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
- `total_words` > 2,000,000 OR `total_files` > 200 → show the warning + top 5 subdirectories by file count. For a verified existing-corpus update, ask whether to continue with the full corpus or stop; never replace it with a narrower corpus. Otherwise ask which subfolder; wait before proceeding.
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

Process only files in `graphify-out/.graphify_uncached.txt`; all cached → skip chunk extraction and cache saves, materialize `.graphify_semantic.json` from this run's cache hits only, then go to Part C. Never include a stale `.graphify_semantic_new.json`:

<!-- script: see references/scripts.md §Step B3 - Materialize cache hits -->

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

`build_from_json()` takes `directed=IS_DIRECTED` — `True` if `--directed` was given, else `False`, except for updates as specified below. `ALLOW_UPDATE_SHRINK` defaults to `False`; the `--update` section sets it on the verified update path.

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

## For --update (repair on change)

A complete fresh extraction on change avoids grafting deltas onto the lossy exported graph; cached semantic results are reused and unchanged runs still stop early. Identify the original corpus root independently of the requested path: use reliable session evidence or the root in `graphify-out/GRAPH_REPORT.md`'s `Graph Report - <root>` heading when it can be resolved unambiguously against the original build's working directory. If the report is missing, stale, or its relative path cannot be resolved reliably, ask the user for the original root before detection. Compare any requested path to that root after resolving both paths; a different root is a separate build, never an update of this graph.

Run incremental detection. No changed or deleted files → stop. Snapshot the published graph to a run-local `<old-graph>` — on snapshot failure, stop before publishing. Read its node-link `directed` boolean; use it for `IS_DIRECTED` throughout Steps 4–9 unless the user explicitly requests a direction change. Missing or invalid direction → stop and report. Run Steps 2–3 on the full current corpus with the semantic cache. If any required input was skipped or failed, stop before Step 4 and leave the published graph and manifest unchanged. Otherwise enable `ALLOW_UPDATE_SHRINK` and run Steps 4–9; publish a smaller nonempty graph only through the staged export — leave the existing graph intact on failed export. Diff after successful Step 4; on success or failure remove only the run-created `<old-graph>` file.

<!-- script: see references/scripts.md §--update - Detect incremental changes -->

<!-- script: see references/scripts.md §--update - Save old graph for diff -->

Set `ALLOW_UPDATE_SHRINK` = `True` in Step 4 only on this verified update path — complete extraction with no skipped or failed required inputs plus a verified deletion or changed content; default `False`. On `True`, Step 4 exports to a run-local temp in the same filesystem and atomically replaces `graphify-out/graph.json` on success; the empty-graph guard still applies.

After successful Step 4, diff the saved and published graphs:

<!-- script: see references/scripts.md §--update - Graph diff -->

---

## For --cluster-only

Skip Steps 1-3. Load `graphify-out/graph.json`, re-run clustering:

<!-- script: see references/scripts.md §--cluster-only -->

Then Steps 5-9 as normal.

---

## For /graphify add

Fetch a URL, add to the corpus, update the graph:

<!-- script: see references/scripts.md §/graphify add - Fetch URL and add to corpus -->

Set `AUTHOR`/`CONTRIBUTOR` to the user's name when provided. Resolve the existing graph's corpus root as in `--update`, or ask before fetching. Save under `<corpus-root>/raw`, then run `--update` on `<corpus-root>` — do not rebuild from `./raw` alone.

URL types (auto-detected): Twitter/X → oEmbed → `.md` · arXiv → abstract+metadata `.md` · PDF → `.pdf` · images → downloaded, vision extraction next build · any webpage → html2text → markdown.
