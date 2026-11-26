---
name: implement
description: "Implement a ticket or planned change on an existing feature/fix branch, or on master."
---

## Required `<project-root>/docs` reads

Shell-read before implementing: `<project-root>/docs/code-navigation.md`, `<project-root>/docs/git-hosts.md`; also read `<project-root>/docs/issue-trackers.md` when a ticket is supplied. Specs may be gitignored — missing → note gap, never invent.

Use git host operations (`<project-root>/docs/git-hosts.md`) for branch check and pull latest; all git host tools need `repo_path`.

1. **Capture Input**: Explicit ticket reference/URL → `<ticket-url>`. Otherwise, if the request explicitly selects an approved plan, use that plan. If neither is supplied, use the unambiguous ticket targeted by the session's current work. Ambiguous → ask; a ticket mentioned without a resolvable reference → ask for it. Without a ticket, use the approved plan in the request or conversation; no plan → ask for it.

2. **Load Planning Context**: When `<ticket-url>` exists:
1. Reuse current in-session evidence covering the primary ticket's requirements, decisions, and patch status and any linked evidence needed to implement them, including notes this session posted. Skip older comments only when a current authoritative spec or reviewed decision summary establishes coverage of prior requirements and decisions.
2. Load uncovered evidence inline via the issue tracker tool (`issue-trackers.md`): list tracker tools → confirm a ticket-load tool exists; missing → `BLOCKED` with `NO_MCP_ACCESS` only while material gaps remain. Read the primary ticket's body and checklists and comments needed to establish current decisions and patch status. Without authoritative coverage of prior decisions, inspect the remaining comments for requirements and supersessions. Apply the same rule to a source ticket (identified by `refUrl`, links, or the body) or other implementation-relevant linked ticket when its decisions inform the work. Read attachments and other linked tickets only when needed to establish the original problem, requirements, decisions, or supersession → `<attachment-insights>`. Limit nested traversal to links that inform implementation; record material gaps when the available evidence cannot establish current decisions.
3. After either reuse or loading, establish whether proposed patches were superseded by a later spec revision; a supersession without a replacement removes the patch. If current status cannot be established, record the gap and do not apply the unconfirmed patch. Derive requirements from the supported ticket evidence. Use linked evidence to clarify requirements without expanding scope.
4. Anchor `<spec-intent>` in the original problem, requested behavior, acceptance criteria, and explicit user decisions in the primary ticket and linked sources. Distinguish these requirements from proposed targets, patches, and implementation mechanics, including those in Solution or checklists. Linked source unavailable → note the gap and use the primary ticket's intent.

Returns (distilled): `<ticket-name>`, `<ticket-url>`, `<ticket-id>`, `<ticket-title>`, `<ticket-summary>`, `<ticket-board>`/`<ticket-websiteId>`, `<ticket-list>`/`<ticket-state>`, `<feature-area>`, `<spec-intent>`, `<acceptance-criteria>`, `<related-entities>`, `<blockers>`, `<gaps>`, `<requirement-items>` (parsed, not yet a todo list).

Primary ticket load fails or `BLOCKED` → report and stop. Without a ticket, derive the applicable fields from the approved plan, leaving ticket-only fields empty. Set `<work-title>`/`<work-summary>` from the ticket title/summary or, without a ticket, the approved plan's title/scope. An approved plan or planning-phase ticket needs no further plan approval.

3. **Detect & Use Working Branch**: Load branch + status → `<current-branch>`; check whether the current feature/fix branch relates to `<work-title>`/`<work-summary>`. Relevance uses the work title/summary slug, not the ticket id — same derivation as `new-branch`.

**Hard rules — no exceptions:**
- NEVER create a branch — no `git checkout -b`, no branch-create tool, no `new-branch` invocation. Branch creation is the user's job via `new-branch`, invoked separately.
- NEVER check out a branch other than `master`/`main` — no searching local/remote for matches, no switching to an existing feature/fix branch even if it matches.
- The only permitted switch is to `master`/`main`, only when the current branch is unrelated.

**Resolution:**
- Relevant: current branch slug matches a slug from `<work-title>`/`<work-summary>`, or falls under `feature/*`|`fix/*`|`feat/*` with a work-derived slug → stay. If an upstream is configured, fetch it — behind → fast-forward it yourself; ahead → stay, the unpushed commits ship via `new-branch`/`ship-changes`; diverged → `git rebase @{u}` only when the replay range is linear — merge commits can carry edits a clean replay silently drops → a merge in `@{u}..HEAD` → STOP; a clean replay (already-upstream commits drop on their own) → synced; conflict → `git rebase --abort`, STOP and report. Without an upstream, skip upstream sync without creating or publishing a remote branch. In either case, behind its base → pull/rebase onto it yourself — behind alone never blocks.
- Current is `master`/`main` → stay.
- Detached `HEAD` — fetch per `git-hosts.md`, then: `HEAD` == `origin/<default>` → stay, it counts as on the base · clean and behind → advance to the tip with a detached checkout · otherwise → stay, the commits belong to the workspace and ship via `new-branch`/`ship-changes`. Never switch a detached `HEAD` to `master`/`main` — a linked shared-git worktree may be unable to, and switching orphans workspace commits.
- Unrelated → switch to `master`/`main`.
- Uncommitted changes → STOP; ask user to stash/commit first — never discard.
- Working branch is `master`/`main` only → `git pull origin master`.
- Confirm clean state before proceeding.

4. **Build Todo List**: derive deliverables from `<spec-intent>`, `<requirement-items>`, `<acceptance-criteria>`, and checklists. Retain the source item IDs for traceability; group related edits serving one outcome. Record proposed targets and patches, including any ticket patches comment, as starting points rather than binding instructions.

**Todo format** — `id` (inherited or `T1`, `T2`, …), `content` (short imperative naming the deliverable), `file` (proposed or `TBD`), `symbol`, `location`, `patch` (if useful), `status` (`pending`).

Rules: every required outcome maps to a deliverable; order by dependency; no workflow items (context, validation, presentation, commit); mark `in_progress` on start, `completed` once verified — never `in_progress` across context boundaries. Missing requirement → add `pending` with justification; revise the list when the approach changes.

5. **Gather Codebase Context**: inspect the affected path, relevant callers, conventions, proposed targets, and premise (the bug path, invariant, or behavior `<spec-intent>` assumes). Resolve targets against current code and report material premise drift → `<context-files>`.

`BLOCKED` → `grep` model/route/feature-flag names, `find_file_by_name` for controllers/repositories/resources, code navigation (`code-navigation.md`); read 1-2 key files; resolve `TBD` items → `<context-files>`.

**Evaluate the solution:** compare the proposed mechanics with the current code and the simplest viable alternative. If a simpler approach satisfies every required outcome and explicit user constraint, use it without further approval, even when it changes targets, patches, or removes unnecessary planned steps. Update the todo list and retain a mapping from each required outcome to its implementation or validation. Report substantive deviations and why they preserve intent; do not silently discard requirements or sync the ticket unless asked.

6. **Resolve Open Questions (optional)**: grill the user ONLY when at least one holds — pre-implementation or on Step-7 loop-back:
- A target needed by the chosen approach cannot be resolved unambiguously.
- `<gaps>`/`<blockers>` from Step 2 are non-empty and material.
- Spec requirements contradict each other — no implementation satisfies all as written.
- A material product or design trade-off requires a user decision; implementation shapes alone do not.
- The chosen change conflicts with an existing invariant, lifecycle hook, or convention and the correct resolution is unclear.
- Premise validation — Step 5 or surfaced mid-edit — shows the spec's assumed bug path, invariant, or expected behavior no longer holds and the right outcome or resolution is unclear.

None hold → skip to Step 7; don't manufacture questions.

Grilling per `grill-with-codebase`. Record each decision into the affected todo item (`file`/`symbol`/`location`/`patch` or a one-line note). Re-sync the todo list, not the ticket card. Stop when every question resolves; don't re-open settled decisions.

7. **Implement on Working Branch**: focused, minimal changes on the resolved branch. `<spec-intent>` bounds the change — unrelated improvements, refactors, or hardening get flagged in the report, not implemented. Drive off `<todo-list>`: next unimplemented `pending` → `in_progress` → implement the chosen approach (a planned patch is a starting point, not a requirement). Verify immediately when useful; otherwise return the item to `pending` with an implemented-awaiting-verification note and proceed to the next unimplemented item. Batch checks for related deliverables after implementation; mark each `completed` only when its outcome is verified. Failed or unverified items remain pending and are reported. Fresh premise drift mid-edit → reevaluate the approach in Step 5 and reconcile the active todo, then resolve any material open questions in Step 6 before continuing. Exactly one `in_progress` at a time. Run focused checks as needed and relevant project checks after implementation.

**Test items**: use existing test conventions and cover the required behavior; adapt proposed test targets and patches to the chosen approach. Include relevant lint and tests in the batch verification.

8. **Audit & Present Implementation**: reconcile the full diff against `<spec-intent>` both ways — every required outcome demonstrated or flagged, and every diff hunk tracing to a deliverable (untraced → revert or flag); an implementation miss → Step 7 (drift → Step 5). Failed or unavailable checks leave affected items pending; report the gap rather than repeating unchanged checks. Report substantive departures from proposed mechanics and why they still satisfy intent. Final report per `code-work` → *Output Discipline* — changed files (`path:line`), validation result, unfinished items, status + diff stat (git host change scan; `git-hosts.md`).

CRITICAL: STOP here. Never commit, push, or request approval — the user ships via commit-and-push / ship-changes.
