---
name: triage
description: "Use when a ticket needs investigation, debugging, or a response. READ-ONLY — no code changes."
---

## Required `<project-root>/docs` reads

Shell-read before investigating the ticket; may be gitignored — missing → note gap, never invent: `<project-root>/docs/code-navigation.md`, `<project-root>/docs/issue-trackers.md`; debug branch only — `<project-root>/docs/bug-trackers.md`, `<project-root>/docs/database-tools.md`, `<project-root>/docs/doc-lookup.md`, `<project-root>/docs/git-hosts.md`, `<project-root>/docs/observability-tools.md`

Load ticket context, determine mode (ask or debug), gather evidence, sync the result back. READ-ONLY — no code changes, no commits.

## Additional Context

Treat ticket systems generically. `<additional-context>` shapes focus. Update existing tickets; create replacements only when asked.

## The Discipline

**Determining the mode is the skill.** Read the ticket, then decide: question to answer or issue to debug?

### Blockers

Escalate blockers (missing context, ambiguity, inaccessible resource, unclear requirements) in order:

1. **Ask the user first** — most blockers resolve in seconds. Ask before posting anything to the ticket.
2. **Post to the ticket only when the user cannot resolve it** — needs another team, customer-side info, or a decision outside the user's authority.

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

Ambiguous → ask the user before posting. Both apply → debug (root cause answers the question too). Store as `<triage-mode>`.

### Gather Evidence

**ask:** entry points, callers, conventions via `grep`/`find_file_by_name`/code navigation (`code-navigation.md`) → `<repo-context>`; doc lookup for API/version questions (`doc-lookup.md`) → `<doc-context>`.

**debug:** run context-gathering — code navigation for symbols, database tools for data (live DB is ground truth), doc lookup for framework docs. Beyond code and data: error monitoring for matching reports/stack traces (`bug-trackers.md`); monitoring/incident platforms (`observability-tools.md`) for alerts overlapping the timeframe, scope, and symptoms — a correlated incident may be the cause, not just a clue. Check recent history: `git log`/`git blame`, merges, releases, deploys touching the affected path (`git-hosts.md`) — a regression's culprit beats a forward fix. Scan `<project-root>/docs/` for other configured tools the issue touches. Trace user action → failure; find where actual diverges from expected → `<investigation-findings>`.

### Production Data for Local Reproduction

You may copy production data into a local database as fixture setup when reproducing a production scenario. The READ-ONLY boundary permits this local setup; code and production data stay unchanged.

- Follow project database-tool and data-handling rules (`database-tools.md`). Confirm the source is production and the destination is local before importing.
- Copy only the records needed for the scenario, including required relationships and tenant context. Preserve reproduction-critical values; mask unrelated sensitive data and exclude secrets from logs, reports, and version control.
- Isolate local mail, queues, webhooks, and external integrations before exercising copied data so reproduction cannot affect real users or services.
- Ask before overwriting or deleting existing local data. Use the imported records to replay the reported scenario and verify local persisted effects; never write to production.

### Findings

Two output shapes — nothing else:

- **ask** → the answer. Direct, one or two sentences; a `file:line` citation only when it proves the answer.
- **debug** → `**Root cause:**` what diverges and why, with `file:line` proof and the culprit commit/deploy when identifiable. Then `**Fixes:**` possible fixes, ranked; revert first when a culprit is confirmed. No implementing.

Done when: reproduced · cause proven with evidence · repro conditions and impact scope known · culprit identified or ruled out. Name what stays unconfirmed in one line, only when it changes the next action.

**Debug Instrumentation** — only when the user grants write access for diagnosis: tag debug logs with a unique prefix `[DEBUG-a4f2]` (cleanup = one grep); one variable at a time; targeted logs at boundaries — never "log everything and grep".

### Sync

Always sync findings back — the spec phase reads prior comments; skipping breaks the pipeline. Via issue tracker sync — convention per `issue-trackers.md` (`refUrl` = `<ticket-url>`). Write the note per `writing-for-agents`: compact, `file:line` citations, no code blocks, no client-facing tone. Sync fails → report error, provide findings in chat.

## Notes

- **Traceability:** when `<question>`/`<additional-context>` came from skill arguments and isn't visible on the ticket, open the findings with one `Question:`/`Context:` line.
- Final output: ticket URL + the findings themselves — the shapes above, nothing more.
