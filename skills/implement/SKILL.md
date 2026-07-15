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

Returns (distilled, no raw card body):
- `<ticket-name>`, `<ticket-url>`
- `<ticket-board>` / `<ticket-websiteId>`, `<ticket-list>` / `<ticket-state>`
- `<feature-area>`: module/domain (e.g., Student, Finance, Report)
- `<acceptance-criteria>`: concrete must-haves from checklists/description
- `<related-entities>`: model names, table names, feature flags
- `<blockers>`: anything marked blocking/waiting
- `<gaps>`: vague/missing requirements, or `None identified`
- `<todo-list>`: structured todo list (format below)

Planning-phase tickets are approved plans — proceed to implementation. No design/plan approval.

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
- State transitions immediate: mark `in_progress` when starting the code change; mark `completed` once verified (pint + tests green for that item). Never batch completions; never leave `in_progress` across context boundaries
- Scope changes explicit: missing requirement → add new `pending` item with one-line justification. Don't silently expand scope; don't drop items without recording why
- Re-sync only on drift: card updates mid-implementation → reconcile todo list before continuing. Todo list mirrors card, not replaces it

If ticket load fails or returns `BLOCKED` (including `NO_MCP_ACCESS`): list tools on the issue tracker, then load the ticket, ingest attachments, analyze requirements, build todo list per rules above.

4. **Gather Codebase Context**: Pass `<feature-area>`, `<related-entities>`, `<acceptance-criteria>`, module path, **and inherited code-target triples (`file`/`symbol`/`location`) from todo list**. **Validate each inherited target against current code** (confirm file exists, symbol at stated location, patch applies cleanly) and report drift. Only resolve `TBD` targets for non-plan cards. Require distilled brief: entry points, related modules, callers, conventions, file:line citations. Store as `<context-files>`.

If `BLOCKED`:
- `grep` for model names, route names, feature flags
- `find_file_by_name` for controllers, repositories, Filament resources
- code navigation tool (see `/docs/code-navigation.md` in the project root) as alternative
- Read 1-2 key files to understand patterns
- Resolve any `TBD` todo item `file`/`symbol`/`location` now, update todo list
- Store relevant paths as `<context-files>`

5. **Resolve Open Questions (optional)**: After Step 3 and Step 4, doubts may surface — plan/code drift, ambiguous acceptance criteria, conflicting patterns, unresolved `<gaps>`/`<blockers>`, or multiple viable shapes for one todo item. When any remain, grill the user before implementing.

Trigger this step ONLY when at least one holds:
- An inherited code-target triple (`file`/`symbol`/`location`) from Step 4 doesn't validate against current code and the right replacement is ambiguous
- `<gaps>` or `<blockers>` from Step 3 are non-empty and material to the work
- A todo item has more than one reasonable implementation shape and the plan's `patch` doesn't disambiguate
- A planned change conflicts with an existing invariant, lifecycle hook, or convention surfaced in Step 4

If none hold, skip this step — proceed to Step 6. Don't manufacture questions.

Grilling rules (mirror `grill-with-codebase`):
- One question at a time. Wait for the answer before asking the next. Don't batch.
- Each question ships with a one-line recommended answer grounded in code read this session. Cite `file:lineRange` for any code reference.
- If a question can be settled by exploring the codebase further, explore instead of asking. Grill only on decisions the code can't settle.
- When the user proposes a shape that contradicts an existing pattern or invariant, surface the conflict with the decisive snippet before moving on.
- Record each resolved decision back into the affected todo item (`file`/`symbol`/`location`/`patch` or a one-line note). Re-sync the todo list, not the ticket card.

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
