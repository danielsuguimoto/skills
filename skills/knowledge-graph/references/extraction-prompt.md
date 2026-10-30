# Chunk extraction prompt

Referenced by `SKILL.md` Step B2 — substitute `FILE_LIST` with the chunk's file list, then use the block verbatim as the prompt.

```
Read the files listed and extract a knowledge graph fragment.
Output ONLY valid JSON with no commentary: {"nodes": [...], "edges": [...], "hyperedges": [...], "input_tokens": 0, "output_tokens": 0}

Extraction rules:
- EXTRACTED: explicit in source (import, call, citation)
- INFERRED: reasonable inference (shared structure, implied dependency)
- AMBIGUOUS: uncertain — flag it, do not omit
- Code files: extract semantic edges AST cannot find (design patterns, protocol conformance). Do not re-extract imports.
- Doc/paper files: named concepts, entities, citations. Store rationale (WHY decisions) as `rationale` attribute on relevant node. Use `file_type:"rationale"` for concept-like nodes (ideas, principles, mechanisms) and `file_type:"concept"` for named concepts. `file_type` MUST be one of: `code`, `document`, `paper`, `image`, `rationale`, `concept`. When adding `calls` edges: source is caller, target is callee.
- Image files: use vision to understand what the image IS (not just OCR). UI screenshot: layout patterns, design decisions, key elements, purpose. Chart: metric, trend/insight, data source. Tweet/post: claim as node, author, concepts mentioned. Diagram: components and connections. Research figure: what it demonstrates, method, result. Handwritten/whiteboard: ideas and arrows, mark uncertain readings AMBIGUOUS.
- DEEP_MODE (if set): be aggressive with INFERRED edges
- Semantic similarity: if two concepts solve same problem without structural link, add `semantically_similar_to` INFERRED edge (confidence 0.6-0.95). Non-obvious cross-file links only.
- Hyperedges: if 3+ nodes share concept/flow not captured by pairwise edges, add hyperedge. Max 3 per file.
- If file has YAML frontmatter (--- ... ---), copy source_url, captured_at, author, contributor onto every node from that file.
- confidence_score REQUIRED on every edge — never omit, never use 0.5 default:
  - EXTRACTED edges: confidence_score = 1.0 always
  - INFERRED edges: pick exactly ONE value — never 0.5:
    0.95 direct structural evidence (shared data structure, named cross-file reference)
    0.85 strong inference (clear functional alignment, no direct symbol link)
    0.75 reasonable inference (shared problem domain + similar shape, requires interpretation)
    0.65 weak inference (thematically related, no shape evidence)
    0.55 speculative but plausible (surface-level co-occurrence only)
  If no value above fits, mark AMBIGUOUS rather than picking 0.4 or below.
- AMBIGUOUS edges: 0.1-0.3

Schema:
{"nodes":[{"id":"filestem_entityname","label":"Human Readable Name","file_type":"code|document|paper|image|rationale|concept","source_file":"relative/path","source_location":null,"source_url":null,"captured_at":null,"author":null,"contributor":null}],"edges":[{"source":"node_id","target":"node_id","relation":"calls|implements|references|cites|conceptually_related_to|shares_data_with|semantically_similar_to|rationale_for","confidence":"EXTRACTED|INFERRED|AMBIGUOUS","confidence_score":1.0,"source_file":"relative/path","source_location":null,"weight":1.0}],"hyperedges":[{"id":"snake_case_id","label":"Human Readable Label","nodes":["node_id1","node_id2","node_id3"],"relation":"participate_in|implement|form","confidence":"EXTRACTED|INFERRED","confidence_score":0.75,"source_file":"relative/path"}],"input_tokens":0,"output_tokens":0}

Files:
FILE_LIST
```
