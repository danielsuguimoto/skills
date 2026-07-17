---
name: triage
description: "Use when answering a ticket question, debugging an issue, or when a ticket needs investigation or a response. READ-ONLY — no code changes."
---

## Required `/docs` reads

Read these project-root spec files before acting (use shell `cat`/`ls` — they may be in `.gitignore`, invisible to built-in search). Missing file → fall back to native tools, note the gap; never invent contents.

- `/docs/code-navigation.md`
- `/docs/database-tools.md`
- `/docs/doc-lookup.md`
- `/docs/issue-trackers.md`

Load ticket context, determine mode (ask or debug), gather evidence, draft findings, sync back. READ-ONLY — no code changes, no commits.

## Additional Context

- Treat ticket systems generically. Use `<additional-context>` to shape tone, depth, focus. Update existing tickets; create replacements only when asked.

## The Discipline

**Determining the mode is the skill.** Read the ticket, then decide: question to answer or issue to debug?

### Interpret Arguments

- Ticket reference/URL → `<ticket-url>`; extra focus/constraints → `<additional-context>`; no arguments → derive from conversation.
- Do NOT expect a `<question>` in arguments — it lives on the ticket. Only if the caller explicitly adds one → store as `<question>` to override/refine.

### Load Ticket Context

- `<ticket-url>` defined: load via issue tracker tool (see `/docs/issue-trackers.md`) with `source: <ticket-url>`, `comments: true` → `<ticket-context>`. Read all attachments with `relativePath` via `read` (images → describe; documents → extract key info → `<attachment-insights>`). Note gaps if inaccessible.
- Otherwise: treat relevant request/conversation as `<ticket-context>`.
- Missing or unloadable → STOP.

### Determine Mode

Read the ticket body and every comment in order. Classify:

- **ask** — ticket contains a question needing an answer (most recent unanswered question; explicit `<question>` from arguments takes precedence).
- **debug** — ticket reports a bug/error/unexpected behavior needing root-cause investigation. Derive `<issue-description>`, `<expected-behavior>`, `<actual-behavior>`, `<reproduction-steps>`, `<error-messages>`, `<environment-context>`. Don't discard earlier comments.

Ambiguous → ask for clarification. Both apply → default to debug (root-cause answers the question too). Store as `<triage-mode>`.

### Gather Evidence

**ask mode:** Gather distilled brief (entry points, callers, conventions, file:line citations) using `grep`/`find_file_by_name`/code navigation tool (see `/docs/code-navigation.md`). Pass `<question>`, relevant entities, module path → `<repo-context>`. For API/version questions, use doc lookup (see `/docs/doc-lookup.md`) → `<doc-context>`.

**debug mode:** Run context-gathering: code navigation for symbols, database tools (see `/docs/database-tools.md`) for data, doc lookup for framework docs. Trace code path user action → failure; identify where actual diverges from expected. Live database is ground truth — verify schema, counts, sample rows directly; don't rely on seeders/migrations/operations. Store as `<investigation-findings>`.

### Draft Findings

**ask mode:** Answer `<question>` using ticket discussion + `<repo-context>` / `<doc-context>`. Store as `<ticket-findings>`.

**debug mode:** Build a root-cause note via these steps:

**Ranked Hypotheses:** Generate 3–5 falsifiable hypotheses. Format: "If `<X>` is the cause, then `<changing Y>` will make the bug disappear." Show ranked list before concluding. Don't block if AFK.

**Debug Instrumentation:** Tag debug logs with unique prefix `[DEBUG-a4f2]` (cleanup = one grep). Change one variable at a time. Targeted logs at boundaries — never "log everything and grep."

**Analyze Root Cause:** Synthesize all findings to determine `<root-cause>`, `<affected-components>`, `<impact-scope>` (single user / company / all), `<reproduction-conditions>`, `<potential-fixes>` (without implementing). Done when: reproduced · 3–5 falsifiable hypotheses ranked · winning hypothesis tested · root cause/components/scope/conditions stated with evidence · gaps named.

**Shape Findings:** Map `<investigation-findings>` to the published note. Carry every field forward (CONFIDENCE, ALTERNATIVE_HYPOTHESES, GAPS). Fields: `<note-title>`, `<issue-summary>`, `<root-cause-analysis>`, `<confidence>`, `<evidence>`, `<affected-areas>`, `<alternative-hypotheses>`, `<recommendations>`, `<gaps>`. Include file paths, line numbers. Store as `<ticket-findings>`.

### Ticket Framing

See `/docs/issue-trackers.md` for provider-specific framing rules. Internal audience: technical language, cite code paths, no client-facing tone (no "Hi", "Thanks", second-person to customer). Do NOT modify/create/delete files; no git commit or push.

### Sync

Always sync findings back — the spec phase reads prior comments; skipping breaks the pipeline. Sync via issue tracker sync tool (see `/docs/issue-trackers.md`): `refUrl` = `<ticket-url>`, `comments` = array with single markdown string. Debug: combine `<note-title>` (as `##` heading) + structured findings. Ask: pass `<ticket-findings>` directly. Follow provider-specific sync rules. If sync fails, report error and provide findings in chat.

## Notes

- Ticket load tool downloads attachments to `storage/app/mcp/{source}` — read before triaging.
- Private note: technical, concise, cite file:line, no full code blocks.
- Final output: ticket URL (+ note title for debug). Don't paste full findings back.

## Red Flags
**Never:** make code changes — READ-ONLY.
**Always:** publish per provider visibility rules · ground in actual findings · cite file paths and line numbers.
