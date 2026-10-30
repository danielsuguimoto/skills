---
name: triage
description: "Use when answering a ticket question, debugging an issue, or when a ticket needs investigation or a response. READ-ONLY — no code changes."
---

## Required `<project-root>/docs` reads

Read via shell before investigating the ticket — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/bug-trackers.md`
- `<project-root>/docs/code-navigation.md`
- `<project-root>/docs/database-tools.md`
- `<project-root>/docs/doc-lookup.md`
- `<project-root>/docs/git-hosts.md`
- `<project-root>/docs/issue-trackers.md`
- `<project-root>/docs/observability-tools.md`

Load ticket context, determine mode (ask or debug), gather evidence, draft findings, sync back. READ-ONLY — no code changes, no commits.

## Additional Context

Treat ticket systems generically. `<additional-context>` shapes tone, depth, focus. Update existing tickets; create replacements only when asked.

## The Discipline

**Determining the mode is the skill.** Read the ticket, then decide: question to answer or issue to debug?

### Blockers

Escalate blockers (missing context, ambiguity, inaccessible resource, unclear requirements) in order:

1. **Ask the user first** — most blockers resolve in seconds via the interactive question tool when one exists. Ask before posting anything to the ticket.
2. **Post to the ticket only when the user cannot resolve it** — needs another team, customer-side info, or a decision outside the user's authority. A ticket-posted blocker the user could have answered adds a round-trip they must unwind.

### Interpret Arguments

Ticket reference/URL → `<ticket-url>`; extra focus/constraints → `<additional-context>`; none → derive from conversation. No `<question>` expected in arguments — it lives on the ticket; an explicit one → store as `<question>` to override/refine.

### Load Ticket Context

- `<ticket-url>` → load via issue tracker (`<project-root>/docs/issue-trackers.md`) with `source: <ticket-url>`, `comments: true` → `<ticket-context>`. Read all attachments — `storage/app/mcp/{source}` — with `relativePath` via `read` (images → describe; documents → extract key info → `<attachment-insights>`); note inaccessible gaps.
- Otherwise → the request/conversation is `<ticket-context>`.
- Missing/unloadable → ask the user for content or an alternative source; STOP only if they cannot provide it.

### Determine Mode

Read the body and every comment in order. Classify:

- **ask** — a question needing an answer (most recent unanswered; explicit `<question>` from arguments takes precedence).
- **debug** — a bug/error/unexpected behavior needing root cause. Derive `<issue-description>`, `<expected-behavior>`, `<actual-behavior>`, `<reproduction-steps>`, `<error-messages>`, `<environment-context>` — don't discard earlier comments.

Ambiguous → ask the user before posting. Both apply → default to debug (root cause answers the question too). Store as `<triage-mode>`.

### Gather Evidence

**ask:** distilled brief (entry points, callers, conventions, file:line citations) via `grep`/`find_file_by_name`/code navigation (`<project-root>/docs/code-navigation.md`) → `<repo-context>`; doc lookup for API/version questions (`<project-root>/docs/doc-lookup.md`) → `<doc-context>`.

**debug:** run context-gathering — code navigation for symbols, database tools for data, doc lookup for framework docs. Beyond code and data: error monitoring for matching reports/stack traces (`<project-root>/docs/bug-trackers.md`); monitoring/incident platforms such as Sentry or Flare (`<project-root>/docs/observability-tools.md`) for alerts overlapping the timeframe, scope, and symptoms — a correlated incident may be the cause, not just a clue. Check recent history: `git log`/`git blame`, merges, releases, deploys touching the affected path (`<project-root>/docs/git-hosts.md`) — the issue may be a regression where reverting the culprit beats a new fix. Scan the rest of `<project-root>/docs/` for other configured tools the issue touches. Trace user action → failure; find where actual diverges from expected. Live database is ground truth — verify schema, counts, sample rows directly, not seeders/migrations/operations → `<investigation-findings>`.

### Draft Findings

**ask:** answer `<question>` from ticket discussion + `<repo-context>`/`<doc-context>` → `<ticket-findings>`.

**debug:** build a root-cause note:

**Ranked Hypotheses:** 3–5 falsifiable: "If `<X>` is the cause, then `<changing Y>` will make the bug disappear." Include a regression hypothesis ("a recent commit or deploy introduced this") whenever the reporter implies it worked before. Show the ranked list before concluding; don't block if AFK.

**Debug Instrumentation** — only when the user grants write access for diagnosis: tag debug logs with unique prefix `[DEBUG-a4f2]` (cleanup = one grep); one variable at a time; targeted logs at boundaries — never "log everything and grep".

**Analyze Root Cause:** synthesize `<root-cause>`, `<introduced-by>` (culprit commit/merge/deploy if identifiable), `<affected-components>`, `<impact-scope>` (single user/company/all), `<reproduction-conditions>`, `<potential-fixes>` (no implementing). Confirmed `<introduced-by>` → lead recommendations with the revert over a forward fix. Done when: reproduced · 3–5 hypotheses ranked · winner tested · cause/components/scope/conditions stated with evidence · culprit identified or ruled out · gaps named.

**Shape Findings:** map `<investigation-findings>` to the note, carrying every field forward: `<note-title>`, `<issue-summary>`, `<root-cause-analysis>`, `<confidence>`, `<evidence>`, `<affected-areas>`, `<alternative-hypotheses>`, `<recommendations>`, `<gaps>` — with file paths and line numbers → `<ticket-findings>`.

### Ticket Framing

Per `<project-root>/docs/issue-trackers.md`. Private note for internal audience: technical, concise, cite file:line, no full code blocks, no client-facing tone (no "Hi", "Thanks", second-person to customer).

### Sync

Always sync findings back — the spec phase reads prior comments; skipping breaks the pipeline. Via issue tracker sync (`<project-root>/docs/issue-trackers.md`): `refUrl` = `<ticket-url>`, `comments` = array with a single markdown string. Debug: `<note-title>` as `##` heading + structured findings. Ask: pass `<ticket-findings>` directly. Follow provider-specific sync rules. Sync fails → report error, provide findings in chat.

## Notes

- **Traceability (required):** `<question>`/`<additional-context>` from skill arguments is invisible to the ticket reader — the findings MUST restate it (quoted preamble or "Question:"/"Context:" block) before the answer. Skip only when already visible in the ticket body/comments.
- Final output: ticket URL (+ note title for debug). Don't paste full findings back.
