---
name: new-branch
description: "Create a work branch from a description, ticket, issue, or current work."
---

## Required `<project-root>/docs` reads

Shell-read before creating or switching to the branch (reuse session reads): `<project-root>/docs/git-hosts.md`. If a spec is missing, report the gap and use a documented fallback.

# Branch From Argument or Work

Create and switch to a categorized branch. All git/gh operations follow `git-hosts.md`.

| Operation | Tool | CLI fallback |
|-----------|------|--------------|
| Load change state | git host change scan | change-scan fallback per spec (sync first per spec, then ahead) |
| Create branch | git host branch create (`branch_name`), if it does not publish | `git checkout -b <cat>/<slug>` |

### 1. Interpret Arguments

Classify the explicit argument first. A forwarded `<ticket-url>` counts as the ticket argument; a forwarded `<ticket-mode>` = `skip` suppresses session inference — `<arg-mode>` = `none`. Otherwise, without an explicit argument, use the unambiguous ticket targeted by the session's current work; ambiguous → ask. Classify the result → `<arg-mode>` (`description`|`ticket`|`none`) + `<branch-context>`:

- **Description** — free-form text, not a URL/issue link → use directly as `<branch-context>`.
- **Ticket/issue link** — get a ticket brief (see Ticket Brief) scoped to title and purpose — plus labels/type for branch naming only when no `<branch-category>` is forwarded → `<branch-context>`. A forwarded `<ticket-brief-result>` covering that scope → reuse it directly, no second read; a forwarded blocker for the same reference → accept it as the final lookup outcome, never retry. No usable brief → fall back to a slug-relevant fragment (issue number, path slug); note the gap. Do not load the raw ticket inline.
- **None** — no argument or session ticket; `<branch-context>` empty; name derived from git changes (Step 2).

### 2. Load Changes

Always runs — git changes are validated every invocation.

Pre-scan forwarded → use `<current-branch>`, `<uncommitted>`, `<ahead-commits>`, `<change-summary>`, `<branch-category>` directly; skip to Step 3.

Otherwise: (1) load change state per the table → `<current-branch>`, `<uncommitted>`; (2) `<uncommitted>` empty → check ahead — `origin/<current-branch>..<current-branch>`, or `origin/<default>..HEAD` on a detached `HEAD` (sync per `git-hosts.md` first — stale `origin/` refs lie) → `<ahead-commits>`; (3) analyze scope — primary `<uncommitted>`, fallback `<ahead-commits>` — group into logical themes, summarize what and why (not how) → `<change-summary>`.

### 3. Check Blockers

- Nothing to branch from: `<arg-mode>` = `none` AND `<uncommitted>` AND `<ahead-commits>` all empty → STOP: "Nothing to branch from."
- Already on a work branch: `<current-branch>` has a `(feature|fix|refactor|docs|test|chore|feat|bugfix|hotfix|perf|build|ci)/` prefix → STOP: "Branching skipped because the current branch already looks like a work branch."

Note: `<arg-mode>` = `description`/`ticket` → missing git changes do NOT block; the branch is created from the argument and the empty change state is recorded as validation output.

### 4. Determine Category and Slug

`<branch-category>` forwarded from pre-scan → reuse; only generate the slug. Otherwise infer:

- `ticket` → from ticket type/labels first; fall back to `<change-summary>` + `<branch-context>`.
- `description` → from `<branch-context>`; fall back to `<change-summary>`.
- `none` → from `<change-summary>` + `<branch-context>`.

Categories: `feature` (new functionality) · `fix` (bug fixes) · `refactor` (restructure, no behavior change) · `docs` · `test` · `chore` (tooling/deps/maintenance) · `perf` · `build` · `ci`.

Slug: concise kebab-case, 2-4 descriptive words → `<branch-slug>`. Source: `description`/`ticket` → `<branch-context>` (consult `<change-summary>` only to disambiguate); `none` → `<change-summary>` + `<branch-context>`.

### 5. Create and Checkout Branch

`<branch-category>/<branch-slug>` via git host branch create or `git checkout -b` per the table. Base: `<uncommitted>` or `<ahead-commits>` non-empty → current checkout; else `origin/<default>` (fresh from the Step-2 sync) — passed to the branch-create operation only when it accepts a base. Never track the base: the new branch must have no upstream, so a later publish establishes `origin/<new-branch>` — verify after creation. Branch exists → retry once with numeric suffix (`-2`) → `<new-branch>`.

Output the new branch name. Pull or publish only when the user explicitly requests it.

## Ticket Brief

Reuse session evidence that covers the references, scoped answers, and decision status. Otherwise dispatch one isolated, read-only worker with the references, scope, `<project-root>`, these rules, and the target PR when known. The parent receives the brief only. If the worker or spec is unavailable, report the gap without fetching inline.

Worker: read `<project-root>/docs/issue-trackers.md`. Dedupe by tracker and ID; read title, body, checklists, and requested metadata. Read the latest comment page when scope includes behavior or constraints. Read older comments, attachments, and linked tickets only to close a scope gap or settle an unclear decision; this overrides general tracker traversal. For the target PR, return relevant decision references without fetching its feedback. Follow other PR links only for specific evidence the scope needs. Ignore unrelated bundled content; leave tracker state unchanged.

Return cited answers, decision status (confirmed, proposed, or superseded), blockers, and evidence gaps. Cite tickets, comment permalinks, or attachment names and locations; omit raw threads and attachments. Mark answers partial if unseen material could change them. Stop when the scope is supported.
