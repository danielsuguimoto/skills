---
name: implement
description: "Develop from a ticket: load details, gather context, implement on master."
---

## MCP Server Preference

Prefer `git-ops` MCP over `git` CLI (fewer tokens); fall back silently when offline. All `git-ops` tools need `repo_path`. See `AGENTS.md` → Git Conventions.

Operations: branch+uncommitted check (`change_scan` → `git branch --show-current` + `git status --short`), switch to master (`checkout` → `git checkout master`), pull latest (`git pull origin master`).

1. **Ensure Clean Master**: Load branch + status. Store `<current-branch>`.
- Not `master`/`main` → switch to master
- Uncommitted on non-master → STOP, ask user to stash/commit first
- Pull latest: `git pull origin master`
- Confirm clean state before proceeding

2. **Capture Ticket Reference**: Extract `<ticket-url>`.
- Ticket URL → use directly
- Mention without URL → ask for link

3. **Load Ticket + Build Todo List**: Delegate to `ticket-loader` subagent (foreground — MCP tools need approval). Ticket is bulky; subagent ingests via MCP, returns distilled fields. Pass `<ticket-url>` and todo-list rules. `profile: "ticket-loader"`, `is_background: false`.

Subagent does:
1. List tools on the ticket MCP server (see `AGENTS.md` → Ticket Tools) → confirm ticket load tool available. Missing → return `BLOCKED` with `NO_MCP_ACCESS`.
2. Load ticket via the ticket MCP tool with `source: <ticket-url>`.
3. Read every attachment with `relativePath` via `read` (images → describe; documents → extract key requirements → `<attachment-insights>`).
4. Analyze requirements from name, description, comments, checklists, attachment insights.
5. **Inherit plan code-targets.** Ticket produced by the planning phase. `Implementation` checklist items encoded as `S1: <title> — <file> @ <symbol>::<location> — <instruction>`; full `<requirement-item>` blocks (with `patch`) appended to ticket description. Parse both: checklist line gives per-item locator triple (`file`/`symbol`/`location`); description block supplies `patch` snippet and full instruction. Carry forward verbatim into todo list. Don't re-derive code targets here.
6. Build structured todo list from `<acceptance-criteria>` and checklists, one todo per parsed `<requirement-item>`. Checklist item lacking encoded locator triple (legacy/non-plan) → fall back to `content`-only format, flag for code-target resolution in Step 4.

Subagent returns (distilled, no raw card body):
- `<ticket-name>`, `<ticket-url>`
- `<ticket-board>` / `<ticket-websiteId>`, `<ticket-list>` / `<ticket-state>`
- `<feature-area>`: module/domain (e.g., Student, Finance, Report)
- `<acceptance-criteria>`: concrete must-haves from checklists/description
- `<related-entities>`: model names, table names, feature flags
- `<blockers>`: anything marked blocking/waiting
- `<gaps>`: vague/missing requirements, or `None identified`
- `<todo-list>`: structured todo list (format below)

Tickets from the planning phase are approved plans — proceed to implementation. No design/plan approval.

**Todo List Format**: One todo per acceptance criterion/checklist item. Each item:
- `id` — inherited from plan's `<requirement-item>` (e.g., `S1`, `S2`). Non-plan cards: assign `T1`, `T2`, …
- `content`: short imperative phrase naming the deliverable (e.g., `Add Status::ACTIVE guard in Placement::save()`), not a workflow step
- `file`: repo-root-relative path(s). Inherited from plan; `TBD` only for non-plan cards pending Step 4
- `symbol`: function/class/method/component to modify. Inherited from plan
- `location`: exact insertion/modification point (e.g., `inside store() before return redirect()`). Inherited from plan
- `patch`: minimal code snippet or unified diff from plan's `<requirement-item>`, if present. Otherwise omitted
- `status`: `pending` initially

Rules:
- One item = one single-purpose deliverable. Split compound criteria into separate items
- Total coverage: every `<acceptance-criteria>` and checklist entry maps to exactly one todo item. No orphans, no extras
- Order by dependency: schema → model → service → UI → tests
- No workflow items: no entries for context gathering, validation, presentation, or commit
- State transitions immediate: mark `in_progress` when starting code change; mark `completed` once verified (pint + tests green for that item). Never batch completions; never leave `in_progress` across context boundaries
- Scope changes explicit: missing requirement → add new `pending` item with one-line justification. Don't silently expand scope; don't drop items without recording why
- Re-sync only on drift: card updates mid-implementation → reconcile todo list before continuing. Todo list mirrors card, not replaces it

`ticket-loader` unavailable or `BLOCKED` (including `NO_MCP_ACCESS`) → fall back inline: list tools on the ticket MCP server, then load the ticket in parent, ingest attachments, analyze requirements, build todo list per rules above.

4. **Gather Codebase Context**: Delegate to a subagent. Pass `<feature-area>`, `<related-entities>`, `<acceptance-criteria>`, module path, **and inherited code-target triples (`file`/`symbol`/`location`) from todo list**. The subagent must **validate each inherited target against current code** (confirm file exists, symbol at stated location, patch applies cleanly) and report drift. Only resolve `TBD` targets for non-plan cards. Require distilled brief: entry points, related modules, callers, conventions, file:line citations. Store as `<context-files>`.

Subagent unavailable or `BLOCKED` → fall back inline:
- `grep` for model names, route names, feature flags
- `find_file_by_name` for controllers, repositories, Filament resources
- Read 1-2 key files to understand patterns
- Resolve any `TBD` todo item `file`/`symbol`/`location` now, update todo list
- Store relevant paths as `<context-files>`

5. **Implement on Master**: Focused, minimal changes on `master`. Drive off `<todo-list>`: pick next `pending` item, mark `in_progress`, apply change at inherited `file`/`symbol`/`location` (use `patch` as starting delta when present; adapt to drift from Step 4), verify, mark `completed`, move on. Exactly one item `in_progress` at a time. After each meaningful chunk:
- PHP: `kool run php vendor/bin/pint --dirty --format agent` (inline)
- Tests: delegate to a subagent with `kool run test --compact`. Unavailable → inline `kool run test --compact`.

6. **Present Implementation**: Complete and validated → present changes and STOP. No approval, no commit, no push.

Load status + diff stat (MCP `change_scan` or `git status --short` + `git diff --stat`).

CRITICAL: STOP here. Never commit, push, or request approval. User invokes separate skills (commit-and-push, ship-changes) to ship.

## Notes

- The ticket load tool downloads attachments to `storage/app/mcp/{source}`. Read before planning.
- Context gathering + implementation + presentation only. Shipping is separate.

## Red Flags
**Never:**
- Commit or push without explicit user confirmation
- Skip context-gathering before implementing
- Implement beyond ticket scope
- Leave validation steps undone
- Drop or skip a `<todo-list>` item without recording why
- Hold multiple `<todo-list>` items `in_progress` at once

**Always:**
- Gather codebase context before writing code
- Run validate-code-changes after implementation
- Present results and stop — no unsolicited next steps
- Build `<todo-list>` from ticket before implementing; drive every code change off it

