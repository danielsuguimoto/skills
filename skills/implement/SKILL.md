---
name: implement
description: "Activate when implementing a ticket or planned change directly on master."
---

Use the git host tool (see `/docs/git-hosts.md` in the project root). All git host tools need `repo_path`.

Operations: Use git host operations (see `/docs/git-hosts.md` in the project root) for branch check, switch to master, pull latest.

1. **Ensure Clean Master**: Load branch + status. Store `<current-branch>`.
- Not `master`/`main` → switch to master
- Uncommitted on non-master → STOP, ask user to stash/commit first
- Pull latest: `git pull origin master`
- Confirm clean state before proceeding

2. **Capture Ticket Reference**: Extract `<ticket-url>`.
- Ticket URL → use directly
- Mention without URL → ask for link

3. **Load Ticket + Build Todo List**: Load the ticket via the issue tracker tool (see `/docs/issue-trackers.md` in the project root). Ticket is bulky. Pass `<ticket-url>` and todo-list rules.

Steps:
1. List tools on the issue tracker (see `/docs/issue-trackers.md` in the project root) → confirm ticket load tool available. Missing → return `BLOCKED` with `NO_MCP_ACCESS`.
2. Load ticket via the issue tracker tool with `source: <ticket-url>`.
3. Read every attachment with `relativePath` via `read` (images → describe; documents → extract key requirements → `<attachment-insights>`).
4. Analyze requirements from name, description, comments, checklists, attachment insights.
5. **Inherit plan code-targets.** Ticket produced by the planning phase. `Implementation` checklist items encoded as `S1: <title> — <file> @ <symbol>::<location> — <instruction>`; full `<requirement-item>` blocks (with `patch`) appended to ticket description. Parse both: checklist line gives per-item locator triple (`file`/`symbol`/`location`); description block supplies `patch` snippet and full instruction. Carry forward verbatim into todo list. Don't re-derive code targets here.
6. Build structured todo list from `<acceptance-criteria>` and checklists, one todo per parsed `<requirement-item>`. Checklist item lacking encoded locator triple (legacy/non-plan) → fall back to `content`-only format, flag for code-target resolution in Step 4.

Returns (distilled): `<ticket-name>`, `<ticket-url>`, `<ticket-board>`/`<ticket-websiteId>`, `<ticket-list>`/`<ticket-state>`, `<feature-area>`, `<acceptance-criteria>`, `<related-entities>`, `<blockers>`, `<gaps>`, `<todo-list>`.

Planning-phase tickets are approved plans — proceed to implementation. No design/plan approval.

**Todo List Format**: One todo per acceptance criterion/checklist item. Each item: `id` (inherited from plan or `T1`, `T2`, …), `content` (short imperative naming the deliverable), `file` (inherited or `TBD`), `symbol` (inherited), `location` (exact insertion point), `patch` (from plan if present), `status` (`pending`).

Rules: one item = one deliverable (split compound criteria). Total coverage: every acceptance criterion maps to exactly one item. Order by dependency: schema → model → service → UI → tests. No workflow items (context gathering, validation, presentation, commit). Mark `in_progress` when starting, `completed` once verified — never batch, never leave `in_progress` across context boundaries. Scope changes explicit: missing requirement → add `pending` with justification. Re-sync only on drift.

If ticket load fails or returns `BLOCKED`: list tools, load ticket, ingest attachments, analyze requirements, build todo list per rules above.

4. **Gather Codebase Context**: Pass `<feature-area>`, `<related-entities>`, `<acceptance-criteria>`, module path, and inherited code-target triples. Validate each inherited target against current code (file exists, symbol at location, patch applies) and report drift. Only resolve `TBD` targets for non-plan cards. Require distilled brief: entry points, related modules, callers, conventions, file:line citations. Store as `<context-files>`.

If `BLOCKED`: `grep` for model/route/feature-flag names, `find_file_by_name` for controllers/repositories/resources, code navigation tool (see `/docs/code-navigation.md`) as alternative. Read 1-2 key files. Resolve `TBD` items, update todo list. Store paths as `<context-files>`.

5. **Resolve Open Questions (optional)**: After Step 3 and Step 4, doubts may surface — plan/code drift, ambiguous acceptance criteria, conflicting patterns, unresolved `<gaps>`/`<blockers>`, or multiple viable shapes for one todo item. When any remain, grill the user before implementing.

Trigger this step ONLY when at least one holds:
- An inherited code-target triple (`file`/`symbol`/`location`) from Step 4 doesn't validate against current code and the right replacement is ambiguous
- `<gaps>` or `<blockers>` from Step 3 are non-empty and material to the work
- A todo item has more than one reasonable implementation shape and the plan's `patch` doesn't disambiguate
- A planned change conflicts with an existing invariant, lifecycle hook, or convention surfaced in Step 4

If none hold, skip this step — proceed to Step 6. Don't manufacture questions.

Grilling rules: follow `grill-with-codebase` discipline (one question at a time with a recommended answer, ground in code, cite `file:lineRange`, explore before asking). When the user proposes a shape that contradicts an existing pattern or invariant, surface the conflict with the decisive snippet before moving on. Record each resolved decision back into the affected todo item (`file`/`symbol`/`location`/`patch` or a one-line note). Re-sync the todo list, not the ticket card.

Stop grilling when every open question resolves. Don't re-open settled decisions. Proceed to Step 6.

6. **Implement on Master**: Focused, minimal changes on `master`. Drive off `<todo-list>`: pick next `pending` item, mark `in_progress`, apply change at inherited `file`/`symbol`/`location` (use `patch` as starting delta when present; adapt to drift from Step 4 or resolution from Step 5), verify, mark `completed`, move on. Exactly one item `in_progress` at a time. After each meaningful chunk:
- PHP: Run the project's lint command (see project config or `/docs/` in the project root).
- Tests: Run the project's test command (see project config or `/docs/` in the project root).

**Test todo items**: when the `in_progress` item is a test, scout existing tests, base classes, factories, and assertion style. Write the test using the inherited `file`/`symbol`/`location`/`patch`, the production code under test, and a distilled brief of conventions from Step 4. Verify: lint + run tests per the bullets above.

7. **Present Implementation**: Complete and validated → present changes and STOP. No approval, no commit, no push.

Load status + diff stat (git host change scan; see `/docs/git-hosts.md` in the project root).

CRITICAL: STOP here. Never commit, push, or request approval. User invokes separate skills (commit-and-push, ship-changes) to ship.

## Notes

- The ticket load tool downloads attachments to the issue tracker's attachment download location (see `/docs/issue-trackers.md` in the project root). Read before planning.
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
