---
name: implement
description: "Activate when implementing a ticket or planned change on an existing feature/fix branch, falling back to master."
---

## Required `<project-root>/docs` reads

Read via shell before implementing the ticket or planned change — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/code-navigation.md`
- `<project-root>/docs/git-hosts.md`
- `<project-root>/docs/issue-trackers.md`

Use git host operations (`<project-root>/docs/git-hosts.md`) for branch check and pull latest; all git host tools need `repo_path`.

1. **Capture Ticket Reference**: Extract `<ticket-url>` from the user's input (URL → use directly; mention without URL → ask for link).

2. **Load Ticket**: Load the ticket via the issue tracker tool (`<project-root>/docs/issue-trackers.md`). Pass `<ticket-url>`.

Steps:
1. List tools on the issue tracker (`<project-root>/docs/issue-trackers.md`) → confirm ticket load tool available. Missing → return `BLOCKED` with `NO_MCP_ACCESS`.
2. Load ticket via the issue tracker tool with `source: <ticket-url>`.
3. Read every attachment with `relativePath` via `read` (images → describe; documents → extract key requirements → `<attachment-insights>`).
4. Analyze requirements from name, description, comments, checklists, attachment insights.

Returns (distilled): `<ticket-name>`, `<ticket-url>`, `<ticket-id>`, `<ticket-title>`, `<ticket-summary>`, `<ticket-board>`/`<ticket-websiteId>`, `<ticket-list>`/`<ticket-state>`, `<feature-area>`, `<acceptance-criteria>`, `<related-entities>`, `<blockers>`, `<gaps>`, `<requirement-items>` (parsed but not yet built into a todo list).

Planning-phase tickets are approved plans — proceed to implementation. No design/plan approval.

If ticket load fails or returns `BLOCKED`: report the failure and stop.

3. **Detect & Use Working Branch**: Using the loaded ticket's `<ticket-title>` and `<ticket-summary>`, load branch + status, store `<current-branch>`, and check whether the **current** branch is already a feature/fix branch related to this ticket. Relevance test uses ticket content (title/summary slug), not the ticket id — same derivation as the `new-branch` skill.

**Hard rules — no exceptions:**
- NEVER create a new branch. No `git checkout -b`, no branch-create tool, no invoking `new-branch`. Branch creation is the user's job via the `new-branch` skill, invoked separately.
- NEVER check out a branch other than `master`/`main`. Do not search local or remote for matching branches. Do not switch to an existing feature/fix branch even if it appears to match the ticket.
- The only branch you may switch to is `master`/`main`, and only when the current branch is unrelated to the ticket.

**Resolution:**
- Relevance test: the current branch name's slug matches a slug derived from `<ticket-title>` or `<ticket-summary>`, or the current branch falls under `feature/*`|`fix/*`|`feat/*` with a slug derived from the ticket content.
- Current branch is a relevant feature/fix branch → stay on it; this is the working branch. Pull/rebase onto its base only if behind.
- Current branch is `master`/`main` → stay on it as the working branch.
- Current branch is unrelated to the ticket → switch to `master`/`main`; that becomes the working branch.
- Uncommitted changes on the current branch → STOP, ask user to stash/commit first; do not discard.
- Only when the working branch is `master`/`main`: pull latest `git pull origin master`.
- Confirm clean state on the resolved working branch before proceeding.

4. **Build Todo List**: From the loaded ticket's `<requirement-items>`, `<acceptance-criteria>`, and checklists, build the structured todo list.
- **Inherit plan code-targets.** Ticket produced by the planning phase. `Implementation` checklist items encoded as `S1: <title> — <file> @ <symbol>::<location> — <instruction>`; full `<requirement-item>` blocks (with `patch`) appended to ticket description. Parse both: checklist line gives per-item locator triple (`file`/`symbol`/`location`); description block supplies `patch` snippet and full instruction. Carry forward verbatim into todo list. Don't re-derive code targets here.
- Build one todo per parsed `<requirement-item>`. Checklist item lacking encoded locator triple (legacy/non-plan) → fall back to `content`-only format, flag for code-target resolution in Step 5.

**Todo List Format**: One todo per acceptance criterion/checklist item. Each item: `id` (inherited from plan or `T1`, `T2`, …), `content` (short imperative naming the deliverable), `file` (inherited or `TBD`), `symbol` (inherited), `location` (exact insertion point), `patch` (from plan if present), `status` (`pending`).

Rules: one item = one deliverable (split compound criteria). Total coverage: every acceptance criterion maps to exactly one item. Order by dependency: schema → model → service → UI → tests. No workflow items (context gathering, validation, presentation, commit). Mark `in_progress` when starting, `completed` once verified — never leave `in_progress` across context boundaries. Scope changes explicit: missing requirement → add `pending` with justification. Re-sync only on drift.

5. **Gather Codebase Context**: Pass `<feature-area>`, `<related-entities>`, `<acceptance-criteria>`, module path, and inherited code-target triples. Validate each inherited target against current code (file exists, symbol at location, patch applies) and report drift. Only resolve `TBD` targets for non-plan cards. Require distilled brief: entry points, related modules, callers, conventions, file:line citations. Store as `<context-files>`.

If `BLOCKED`: `grep` for model/route/feature-flag names, `find_file_by_name` for controllers/repositories/resources, code navigation tool (`<project-root>/docs/code-navigation.md`) as alternative. Read 1-2 key files. Resolve `TBD` items, update todo list. Store paths as `<context-files>`.

6. **Resolve Open Questions (optional)**: After Step 2 and Step 5, doubts may surface — plan/code drift, ambiguous acceptance criteria, conflicting patterns, unresolved `<gaps>`/`<blockers>`, or multiple viable shapes for one todo item. When any remain, grill the user before implementing.

Trigger this step ONLY when at least one holds:
- An inherited code-target triple (`file`/`symbol`/`location`) from Step 5 doesn't validate against current code and the right replacement is ambiguous
- `<gaps>` or `<blockers>` from Step 2 are non-empty and material to the work
- A todo item has more than one reasonable implementation shape and the plan's `patch` doesn't disambiguate
- A planned change conflicts with an existing invariant, lifecycle hook, or convention surfaced in Step 5

If none hold, skip this step — proceed to Step 7. Don't manufacture questions.

Grilling rules: one question at a time with a recommended answer, ground in code, cite `file:lineRange`, explore before asking. Ask via the environment's interactive question tool when one exists. When the user proposes a shape that contradicts an existing pattern or invariant, surface the conflict with the decisive snippet before moving on. Record each resolved decision back into the affected todo item (`file`/`symbol`/`location`/`patch` or a one-line note). Re-sync the todo list, not the ticket card.

Stop grilling when every open question resolves. Don't re-open settled decisions. Proceed to Step 7.

7. **Implement on Working Branch**: Focused, minimal changes on the resolved working branch (existing feature/fix branch when present, else `master`/`main`). Drive off `<todo-list>`: pick next `pending` item, mark `in_progress`, apply change at inherited `file`/`symbol`/`location` (use `patch` as starting delta when present; adapt to drift from Step 5 or resolution from Step 6), verify, mark `completed`, move on. Exactly one item `in_progress` at a time. After each meaningful chunk, run the project's lint and test commands.

**Test todo items**: when the `in_progress` item is a test, scout existing tests, base classes, factories, and assertion style. Write the test using the inherited `file`/`symbol`/`location`/`patch`, the production code under test, and a distilled brief of conventions from Step 5. Verify: lint + run tests per above.

8. **Present Implementation**: Complete and validated → present changes.

Load status + diff stat (git host change scan; `<project-root>/docs/git-hosts.md`).

CRITICAL: STOP here. Never commit, push, or request approval. User invokes separate skills (commit-and-push, ship-changes) to ship.

## Notes

- Ticket load tool downloads attachments to the provider's download location (`<project-root>/docs/issue-trackers.md`).
