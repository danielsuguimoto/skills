---
name: implement
description: "Activate when implementing a ticket or planned change on an existing feature/fix branch, falling back to master."
---

## Required `<project-root>/docs` reads

Shell-read before implementing: `<project-root>/docs/code-navigation.md`, `<project-root>/docs/git-hosts.md`; also read `<project-root>/docs/issue-trackers.md` when a ticket is supplied. Specs may be gitignored — missing → note gap, never invent.

Use git host operations (`<project-root>/docs/git-hosts.md`) for branch check and pull latest; all git host tools need `repo_path`.

1. **Capture Input**: Extract `<ticket-url>` when supplied (mention without URL → ask for link). Otherwise use the approved plan in the request or conversation; no plan → ask for it.

2. **Load Planning Context**: When `<ticket-url>` exists, use the issue tracker tool (`issue-trackers.md`):
1. List tracker tools → confirm a ticket-load tool exists; missing → return `BLOCKED` with `NO_MCP_ACCESS`.
2. Load the primary ticket inline in the main session. Load its source ticket (identified by `refUrl`, links, or the body) and other implementation-relevant linked tickets inline too. Read their bodies, full comment threads, checklists, and attachments per spec → `<attachment-insights>`. Limit nested traversal to links that inform the implementation.
3. Derive requirements from the primary ticket's name, description, comments, checklists, and attachments. Use linked evidence to clarify requirements without expanding scope.
4. Anchor `<spec-intent>` in the primary ticket's Problem Statement/Solution/Notes and the original problem and decisions in linked sources. Linked source unavailable → note the gap and use the primary ticket's intent. Items are the how; intent arbitrates every later adaptation.

Returns (distilled): `<ticket-name>`, `<ticket-url>`, `<ticket-id>`, `<ticket-title>`, `<ticket-summary>`, `<ticket-board>`/`<ticket-websiteId>`, `<ticket-list>`/`<ticket-state>`, `<feature-area>`, `<spec-intent>`, `<acceptance-criteria>`, `<related-entities>`, `<blockers>`, `<gaps>`, `<requirement-items>` (parsed, not yet a todo list).

Primary ticket load fails or `BLOCKED` → report and stop. Without a ticket, derive the applicable fields from the approved plan, leaving ticket-only fields empty. Set `<work-title>`/`<work-summary>` from the ticket title/summary or, without a ticket, the approved plan's title/scope. An approved plan or planning-phase ticket needs no further plan approval.

3. **Detect & Use Working Branch**: Load branch + status → `<current-branch>`; check whether the current feature/fix branch relates to `<work-title>`/`<work-summary>`. Relevance uses the work title/summary slug, not the ticket id — same derivation as `new-branch`.

**Hard rules — no exceptions:**
- NEVER create a branch — no `git checkout -b`, no branch-create tool, no `new-branch` invocation. Branch creation is the user's job via `new-branch`, invoked separately.
- NEVER check out a branch other than `master`/`main` — no searching local/remote for matches, no switching to an existing feature/fix branch even if it matches.
- The only permitted switch is to `master`/`main`, only when the current branch is unrelated.

**Resolution:**
- Relevant: current branch slug matches a slug from `<work-title>`/`<work-summary>`, or falls under `feature/*`|`fix/*`|`feat/*` with a work-derived slug → stay. If an upstream is configured, fetch it — behind → fast-forward only, diverged → STOP and report. Without an upstream, skip upstream sync without creating or publishing a remote branch. In either case, pull/rebase onto its base only if behind.
- Current is `master`/`main` → stay.
- Detached `HEAD` at the base commit (`HEAD` == `origin/<default>` — fetch first per `git-hosts.md`) → stay; it counts as on the base. Never switch it to `master`/`main` — a linked shared-git worktree may be unable to.
- Unrelated → switch to `master`/`main`.
- Uncommitted changes → STOP; ask user to stash/commit first — never discard.
- Working branch is `master`/`main` only → `git pull origin master`.
- Confirm clean state before proceeding.

4. **Build Todo List**: from `<requirement-items>`, `<acceptance-criteria>`, checklists.
- **Inherit plan code-targets.** Parse locator triples and patch blocks from the approved plan or ticket, including the ticket's patches comment when present. Carry known `file`/`symbol`/`location` and `patch` verbatim. Flag items without targets for resolution in Step 5.

**Todo format** — one per acceptance criterion/checklist item: `id` (inherited or `T1`, `T2`, …), `content` (short imperative naming the deliverable), `file` (inherited or `TBD`), `symbol`, `location`, `patch` (from plan if present), `status` (`pending`).

Rules: one item = one deliverable (split compounds); every criterion maps to exactly one item; order by dependency (schema → model → service → UI → tests); no workflow items (context, validation, presentation, commit); mark `in_progress` on start, `completed` once verified — never `in_progress` across context boundaries; missing requirement → add `pending` with justification; re-sync only on drift.

5. **Gather Codebase Context**: pass `<feature-area>`, `<related-entities>`, `<acceptance-criteria>`, `<spec-intent>`, module path, inherited code-target triples. Validate each inherited target against current code (file exists, symbol at location, patch applies) and premise (the bug path, invariant, or behavior `<spec-intent>` assumes still holds); report target and premise drift. Resolve `TBD` targets only for non-plan cards. Require a distilled brief: entry points, related modules, callers, conventions, file:line citations → `<context-files>`.

`BLOCKED` → `grep` model/route/feature-flag names, `find_file_by_name` for controllers/repositories/resources, code navigation (`code-navigation.md`); read 1-2 key files; resolve `TBD` items → `<context-files>`.

6. **Resolve Open Questions (optional)**: grill the user ONLY when at least one holds — pre-implementation or on Step-7 loop-back:
- An inherited code-target triple fails Step-5 validation and the right replacement is ambiguous.
- `<gaps>`/`<blockers>` from Step 2 are non-empty and material.
- Spec requirements contradict each other — no implementation satisfies all as written.
- A todo item has multiple reasonable shapes the plan's `patch` doesn't disambiguate.
- A planned change conflicts with an existing invariant, lifecycle hook, or convention.
- Premise validation — Step 5 or surfaced mid-edit — shows the spec's assumed bug path, invariant, or expected behavior no longer holds: premise drift, a misdiagnosed root cause, or a superseding change.

None hold → skip to Step 7; don't manufacture questions.

Grilling per `grill-with-codebase`. Record each decision into the affected todo item (`file`/`symbol`/`location`/`patch` or a one-line note). Re-sync the todo list, not the ticket card. Stop when every question resolves; don't re-open settled decisions.

7. **Implement on Working Branch**: focused, minimal changes on the resolved branch. The list bounds the change — improvements, refactors, or hardening beyond what an item requires get flagged in the report, not implemented. Drive off `<todo-list>`: next `pending` → `in_progress` → apply at inherited `file`/`symbol`/`location` (`patch` as starting delta; drift at any point → adapt toward `<spec-intent>`; fresh premise drift surfaced mid-edit → loop back to Step 6) → verify (deliverable observable via focused check, test, or probe demonstrating the criterion) → `completed`. Exactly one `in_progress` at a time. Run focused checks as needed and relevant project checks after implementation.

**Test items**: scout existing tests, base classes, factories, assertion style; write from inherited `file`/`symbol`/`location`/`patch` + production code under test + Step-5 conventions. Verify via lint + tests.

8. **Audit & Present Implementation**: reconcile the full diff against `<spec-intent>` both ways — every acceptance criterion demonstrated or flagged, and every diff hunk tracing to a todo item or criterion (untraced → revert or flag); a miss → Step 7 (drift → Step 6). Clean → final report per `code-work` → *Output Discipline* — changed files (`path:line`), validation result, status + diff stat (git host change scan; `git-hosts.md`).

CRITICAL: STOP here. Never commit, push, or request approval — the user ships via commit-and-push / ship-changes.
