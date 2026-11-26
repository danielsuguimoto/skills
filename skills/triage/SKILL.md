---
name: triage
description: "Investigate, diagnose, or answer a ticket without changing code; route it to a spec or draft reply."
---

## Required `<project-root>/docs` reads

Shell-read before investigating the ticket (reuse session reads): `<project-root>/docs/code-navigation.md`, `<project-root>/docs/issue-trackers.md`; when needed for the evidence — `<project-root>/docs/bug-trackers.md`, `<project-root>/docs/database-tools.md`, `<project-root>/docs/doc-lookup.md`, `<project-root>/docs/observability-tools.md`. If a spec is missing, report the gap and use a documented fallback.

Load ticket context, determine mode (ask or debug), gather evidence, sync the result back. READ-ONLY — no code changes, no commits.

## Additional Context

Treat ticket systems generically. `<additional-context>` shapes focus. Update existing tickets; create replacements only when asked.

## The Discipline

**Determining the mode is the skill.** Read the ticket, then decide: question to answer or issue to debug?

### Blockers

Escalate blockers (missing context, ambiguity, inaccessible resource, unclear requirements) in order:

0. **Resolve it yourself first** — a question the ticket, code, or tools can answer is not a blocker. Exhaust those before escalating.
1. **Ask the user first** — most blockers resolve in seconds. Ask before posting anything to the ticket.
2. **Post to the ticket only when the user cannot resolve it** — needs another team, customer-side info, or a decision outside the user's authority.

### Interpret Arguments

Explicit ticket reference/URL → `<ticket-url>`; otherwise use the unambiguous ticket targeted by the session's current work. Ambiguous → ask; no ticket target → derive context from the conversation. Extra focus/constraints → `<additional-context>`. No `<question>` expected in arguments — it lives on the ticket; an explicit one → store as `<question>` to override/refine.

### Load Ticket Context

- `<ticket-url>` → reuse a complete, current in-session copy — body, every comment including notes this session posted, attachments per spec; otherwise load via issue tracker (`issue-trackers.md`) with `source: <ticket-url>`, `comments: true` → `<ticket-context>`; attachments per spec → `<attachment-insights>`.
- Otherwise → the request/conversation is `<ticket-context>`.
- Missing/unloadable → ask the user for content or an alternative source; STOP only if they cannot provide it.

### Determine Mode

Read the body and every comment in order. Classify:

- **ask** — a question needing an answer (most recent unanswered; explicit `<question>` from arguments takes precedence).
- **debug** — a bug/error/unexpected behavior needing root cause. Derive `<issue-description>`, `<expected-behavior>`, `<actual-behavior>`, `<reproduction-steps>`, `<error-messages>`, `<environment-context>` — don't discard earlier comments.

Ambiguous → ask the user before posting. Both apply → debug (root cause answers the question too). Store as `<triage-mode>`.

### Gather Evidence

**Sync first:** fetch per `git-hosts.md`; the checked-out branch behind → fast-forward a clean tree, dirty/diverged → flag staleness in findings — `file:line` citations on stale code drift.

Gather only evidence needed to answer the question or establish the cause, impact, and reproduction conditions. Start at the reported path; expand to another source only when it can resolve a material unknown. Reuse ticket and session evidence; stop when the finding is supported.

**Cold code scan:** when answering requires multiple searches or reads likely to add distracting intermediate context, and a cited brief suffices, prefer a read-only subagent. Scope it to the question or failure path, known ticket facts and relevant paths; request a concise brief with `file:line` evidence, counterevidence, and gaps. Skip when current context already answers the question. The parent evaluates the brief, checks decisive evidence when needed, and owns the finding; unavailable or insufficient scan → inspect the missing evidence inline.

**ask:** inspect the relevant entry point and callers via code navigation (`code-navigation.md`); use doc lookup for unresolved API/version questions.

**debug:** trace user action → failure and find where actual diverges from expected → `<investigation-findings>`. Use live data when data or schema matters, error monitoring when reports or traces are needed, and observability when an incident is a plausible cause. Use doc lookup for unresolved framework behavior. Independent evidence streams may be gathered in parallel: read-only, three or more needed streams or the user's explicit request, and a suitable subagent profile exists; doubtful independence stays inline. Each returns cited findings for the parent to synthesize — a stream whose results scope another stream runs first. Work the investigation as sequential thinking — branch candidate causes, revise as evidence lands.

### Production Data for Local Reproduction

Explicitly invoking `triage` authorizes the reversible local fixture imports and replay writes needed for reproduction. Once the target and effects are verified, reuse that authorization for each covered write. Keep code and production data unchanged, and honor tool-enforced permissions.

- Follow project database-tool and data-handling rules (`database-tools.md`). Confirm the source is production and the destination is local before importing.
- Copy only the records needed for the scenario, including required relationships and tenant context. Preserve reproduction-critical values; mask unrelated sensitive data and exclude secrets from logs, reports, and version control.
- Isolate local mail, queues, webhooks, and external integrations before exercising copied data so reproduction cannot affect real users or services.
- Ask before overwriting or deleting existing local data. Use the imported records to replay the reported scenario and verify local persisted effects; never write to production.

### Findings

Two output shapes — nothing else:

- **ask** → the answer. Direct, one or two sentences; a `file:line` citation only when it proves the answer.
- **debug** → `**Root cause:**` what diverges and why, with `file:line` proof. Then `**Fixes:**` the simplest viable fix; include alternatives only when a material trade-off needs a decision. No implementing.

Finish when evidence proves the cause and reproduction conditions and impact are known. Name an unconfirmed detail only when it changes the next action.

### Route

The last output line names the follow-up skill — always exactly one:

- `Next: to-specs` — a code change is needed AND spec-ready: root cause proven (or the request unambiguous) and nothing remains that only the client can answer.
- `Next: draft-reply` — no code change needed (answered, not-a-bug, environment/config resolution), OR client input still missing (can't reproduce, missing information, a decision only the requester can make) — the drafted reply delivers the answer or asks the questions.

**Debug Instrumentation** — only when the user grants write access for diagnosis: tag debug logs with a unique prefix `[DEBUG-a4f2]` (cleanup = one grep); one variable at a time; targeted logs at boundaries — never "log everything and grep".

### Sync

Always sync findings back — the spec phase reads prior comments; skipping breaks the pipeline. Explicit invocation covers the findings note on the resolved ticket; do not ask again for that post. A `<ticket-url>` first identified here — not present when the ticket context was loaded → load the ticket thread before posting and confirm the findings still apply; material conflict → re-analyze, never post stale findings. Missing `<ticket-url>` → ask for the target before syncing. Via issue tracker sync — convention per `issue-trackers.md` (`refUrl` = `<ticket-url>`). Write the note per `writing-for-agents`: compact, `file:line` citations, no code blocks, no client-facing tone; the posted note joins `<ticket-context>` for downstream reuse. Sync fails → report error, provide findings in chat.

## Notes

- **Traceability:** when `<question>`/`<additional-context>` came from skill arguments and isn't visible on the ticket, open the findings with one `Question:`/`Context:` line.
- Final output: ticket URL + findings + `Next:` line — nothing more.
