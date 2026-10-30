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

1. **Capture Ticket Reference**: Extract `<ticket-url>` from input (URL → use directly; mention without URL → ask for link).

2. **Load Ticket**: via the issue tracker tool (`<project-root>/docs/issue-trackers.md`):
1. List tracker tools → confirm a ticket-load tool exists; missing → return `BLOCKED` with `NO_MCP_ACCESS`.
2. Load with `source: <ticket-url>`.
3. Read every attachment with `relativePath` via `read` (images → describe; documents → extract key requirements → `<attachment-insights>`).
4. Analyze requirements from name, description, comments, checklists, attachment insights.

Returns (distilled): `<ticket-name>`, `<ticket-url>`, `<ticket-id>`, `<ticket-title>`, `<ticket-summary>`, `<ticket-board>`/`<ticket-websiteId>`, `<ticket-list>`/`<ticket-state>`, `<feature-area>`, `<acceptance-criteria>`, `<related-entities>`, `<blockers>`, `<gaps>`, `<requirement-items>` (parsed, not yet a todo list).

Planning-phase tickets are approved plans — proceed to implementation, no design/plan approval. Ticket load fails or `BLOCKED` → report and stop.

3. **Detect & Use Working Branch**: From `<ticket-title>`/`<ticket-summary>`, load branch + status → `<current-branch>`; check whether the current branch is already a feature/fix branch related to this ticket. Relevance uses ticket content (title/summary slug), not the ticket id — same derivation as `new-branch`.

**Hard rules — no exceptions:**
- NEVER create a branch — no `git checkout -b`, no branch-create tool, no `new-branch` invocation. Branch creation is the user's job via `new-branch`, invoked separately.
- NEVER check out a branch other than `master`/`main` — no searching local/remote for matches, no switching to an existing feature/fix branch even if it matches.
- The only permitted switch is to `master`/`main`, only when the current branch is unrelated.

**Resolution:**
- Relevant: current branch slug matches a slug from `<ticket-title>`/`<ticket-summary>`, or falls under `feature/*`|`fix/*`|`feat/*` with a ticket-derived slug → stay; pull/rebase onto its base only if behind.
- Current is `master`/`main` → stay.
- Unrelated → switch to `master`/`main`.
- Uncommitted changes → STOP; ask user to stash/commit first — never discard.
- Working branch is `master`/`main` only → `git pull origin master`.
- Confirm clean state before proceeding.

4. **Build Todo List**: from `<requirement-items>`, `<acceptance-criteria>`, checklists.
- **Inherit plan code-targets.** `Implementation` checklist items encode `S1: <title> — <file> @ <symbol>::<location> — <instruction>`; full `<requirement-item>` blocks (with `patch`) are appended to the ticket description. Parse both: checklist → locator triple (`file`/`symbol`/`location`); description block → `patch` + full instruction. Carry verbatim — don't re-derive targets. Checklist item lacking the triple (legacy/non-plan) → `content`-only, flag for resolution in Step 5.

**Todo format** — one per acceptance criterion/checklist item: `id` (inherited or `T1`, `T2`, …), `content` (short imperative naming the deliverable), `file` (inherited or `TBD`), `symbol`, `location`, `patch` (from plan if present), `status` (`pending`).

Rules: one item = one deliverable (split compounds); every criterion maps to exactly one item; order by dependency (schema → model → service → UI → tests); no workflow items (context, validation, presentation, commit); mark `in_progress` on start, `completed` once verified — never `in_progress` across context boundaries; missing requirement → add `pending` with justification; re-sync only on drift.

5. **Gather Codebase Context**: pass `<feature-area>`, `<related-entities>`, `<acceptance-criteria>`, module path, inherited code-target triples. Validate each inherited target against current code (file exists, symbol at location, patch applies); report drift. Resolve `TBD` targets only for non-plan cards. Require a distilled brief: entry points, related modules, callers, conventions, file:line citations → `<context-files>`.

`BLOCKED` → `grep` model/route/feature-flag names, `find_file_by_name` for controllers/repositories/resources, code navigation (`<project-root>/docs/code-navigation.md`); read 1-2 key files; resolve `TBD` items → `<context-files>`.

6. **Resolve Open Questions (optional)**: grill the user before implementing ONLY when at least one holds:
- An inherited code-target triple fails Step-5 validation and the right replacement is ambiguous.
- `<gaps>`/`<blockers>` from Step 2 are non-empty and material.
- A todo item has multiple reasonable shapes the plan's `patch` doesn't disambiguate.
- A planned change conflicts with an existing invariant, lifecycle hook, or convention.

None hold → skip to Step 7; don't manufacture questions.

Grilling: one question at a time with a recommended answer, grounded in code, cite `file:lineRange`, explore before asking, interactive question tool when one exists. User proposes a shape contradicting a pattern/invariant → surface the conflict with the decisive snippet. Record each decision into the affected todo item (`file`/`symbol`/`location`/`patch` or a one-line note). Re-sync the todo list, not the ticket card. Stop when every question resolves; don't re-open settled decisions.

7. **Implement on Working Branch**: focused, minimal changes on the resolved branch. Drive off `<todo-list>`: next `pending` → `in_progress` → apply at inherited `file`/`symbol`/`location` (`patch` as starting delta; adapt to Step-5 drift or Step-6 resolution) → verify → `completed`. Exactly one `in_progress` at a time. After each meaningful chunk, run project lint and test commands.

**Test items**: scout existing tests, base classes, factories, assertion style; write from inherited `file`/`symbol`/`location`/`patch` + production code under test + Step-5 conventions. Verify via lint + tests.

8. **Present Implementation**: complete and validated → present changes. Load status + diff stat (git host change scan; `<project-root>/docs/git-hosts.md`).

CRITICAL: STOP here. Never commit, push, or request approval — the user ships via commit-and-push / ship-changes.

## Notes

- Ticket load tool downloads attachments to the provider's download location (`<project-root>/docs/issue-trackers.md`).
