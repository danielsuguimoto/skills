---
name: triage
description: "Use when a ticket needs investigation, debugging, or a response. READ-ONLY — no code changes."
---

## Required `<project-root>/docs` reads

Shell-read before investigating the ticket; may be gitignored — missing → note gap, never invent: `<project-root>/docs/bug-trackers.md`, `<project-root>/docs/code-navigation.md`, `<project-root>/docs/database-tools.md`, `<project-root>/docs/doc-lookup.md`, `<project-root>/docs/git-hosts.md`, `<project-root>/docs/issue-trackers.md`, `<project-root>/docs/observability-tools.md`

Load ticket context, determine mode (ask or debug), gather evidence, draft findings, sync back. READ-ONLY — no code changes, no commits.

## Additional Context

Treat ticket systems generically. `<additional-context>` shapes tone, depth, focus. Update existing tickets; create replacements only when asked.

## The Discipline

**Determining the mode is the skill.** Read the ticket, then decide: question to answer or issue to debug?

### Blockers

Escalate blockers (missing context, ambiguity, inaccessible resource, unclear requirements) in order:

1. **Ask the user first** — most blockers resolve in seconds. Ask before posting anything to the ticket.
2. **Post to the ticket only when the user cannot resolve it** — needs another team, customer-side info, or a decision outside the user's authority. A ticket-posted blocker the user could have answered adds a round-trip they must unwind.

### Interpret Arguments

Ticket reference/URL → `<ticket-url>`; extra focus/constraints → `<additional-context>`; none → derive from conversation. No `<question>` expected in arguments — it lives on the ticket; an explicit one → store as `<question>` to override/refine.

### Load Ticket Context

- `<ticket-url>` → load via issue tracker (`issue-trackers.md`) with `source: <ticket-url>`, `comments: true` → `<ticket-context>`; attachments per spec → `<attachment-insights>`.
- Otherwise → the request/conversation is `<ticket-context>`.
- Missing/unloadable → ask the user for content or an alternative source; STOP only if they cannot provide it.

### Determine Mode

Read the body and every comment in order. Classify:

- **ask** — a question needing an answer (most recent unanswered; explicit `<question>` from arguments takes precedence).
- **debug** — a bug/error/unexpected behavior needing root cause. Derive `<issue-description>`, `<expected-behavior>`, `<actual-behavior>`, `<reproduction-steps>`, `<error-messages>`, `<environment-context>` — don't discard earlier comments.

Ambiguous → ask the user before posting. Both apply → default to debug (root cause answers the question too). Store as `<triage-mode>`.

### Gather Evidence

**ask:** distilled brief (entry points, callers, conventions, file:line citations) via `grep`/`find_file_by_name`/code navigation (`code-navigation.md`) → `<repo-context>`; doc lookup for API/version questions (`doc-lookup.md`) → `<doc-context>`.

**debug:** run context-gathering — code navigation for symbols, database tools for data (live DB is ground truth), doc lookup for framework docs. Beyond code and data: error monitoring for matching reports/stack traces (`bug-trackers.md`); monitoring/incident platforms (`observability-tools.md`) for alerts overlapping the timeframe, scope, and symptoms — a correlated incident may be the cause, not just a clue. Check recent history: `git log`/`git blame`, merges, releases, deploys touching the affected path (`git-hosts.md`) — the issue may be a regression where reverting the culprit beats a new fix. Scan the rest of `<project-root>/docs/` for other configured tools the issue touches. Trace user action → failure; find where actual diverges from expected → `<investigation-findings>`.

### Draft Findings

**ask:** answer `<question>` from ticket discussion + `<repo-context>`/`<doc-context>` → `<ticket-findings>`.

**debug:** build a root-cause note:

**Ranked Hypotheses:** 3–5 falsifiable per `debug` format. Include a regression hypothesis ("a recent commit or deploy introduced this") whenever the reporter implies it worked before. Show the ranked list before concluding; don't block if AFK.

**Debug Instrumentation** — only when the user grants write access for diagnosis: tag debug logs with unique prefix `[DEBUG-a4f2]` (cleanup = one grep); one variable at a time; targeted logs at boundaries — never "log everything and grep".

**Analyze Root Cause:** synthesize `<root-cause>`, `<introduced-by>` (culprit commit/merge/deploy if identifiable), `<affected-components>`, `<impact-scope>` (single user/company/all), `<reproduction-conditions>`, `<potential-fixes>` (no implementing). Confirmed `<introduced-by>` → lead recommendations with the revert over a forward fix. Done when: reproduced · 3–5 hypotheses ranked · winner tested · cause/components/scope/conditions stated with evidence · culprit identified or ruled out · gaps named.

**Shape Findings:** map `<investigation-findings>` to a compact note: `## <note-title>` · `<issue-summary>` (one or two lines) · `<root-cause-analysis>` including `<introduced-by>` when identified · `<evidence>` as `file:line` and query citations · `<confidence>` · `<recommendations>` · `<gaps>`. Add `<affected-areas>` or `<alternative-hypotheses>` only when they change what the reader does next. Omit a field that would be empty — no TBD placeholders → `<ticket-findings>`.

### Ticket Framing

Per `issue-trackers.md`. The note is read by downstream agents (the spec phase reads prior comments) and the team — write it per `writing-for-agents`: compact reference structure, `file:line` citations, no full code blocks, no client-facing tone (no "Hi", "Thanks", second-person to customer).

### Sync

Always sync findings back — the spec phase reads prior comments; skipping breaks the pipeline. Via issue tracker sync — convention per `issue-trackers.md` (`refUrl` = `<ticket-url>`). Debug: `<note-title>` as `##` heading + structured findings. Ask: pass `<ticket-findings>` directly. Sync fails → report error, provide findings in chat.

## Notes

- **Traceability:** when `<question>`/`<additional-context>` came from skill arguments and isn't visible on the ticket, open the findings with one `Question:`/`Context:` line so downstream readers have it.
- Final output: ticket URL (+ note title for debug). Don't paste full findings back.
