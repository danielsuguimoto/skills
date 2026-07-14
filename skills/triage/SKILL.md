---
name: triage
description: "Use when answering a ticket question, debugging an issue, or when a ticket needs investigation or a response. READ-ONLY — no code changes."
---

Load ticket context, determine mode (ask or debug), gather evidence, draft findings, sync back.

CRITICAL: READ-ONLY. Do NOT write, edit, or commit code. Gather context, investigate, post findings only.

## Additional Context

- Treat ticket systems generically. Don't assume a provider unless context makes it relevant.
- Use `<additional-context>` to shape tone, depth, focus.
- For existing tickets, update the same ticket instead of creating a replacement.

## The Discipline

**Determining the mode is the skill.** Read the ticket, then decide: question to answer or issue to debug?

### Interpret Arguments

- Ticket reference/URL → `<ticket-url>`
- Extra focus areas/caveats/constraints → `<additional-context>`
- No arguments → derive request from conversation
- Do NOT expect a `<question>` in arguments. The question or issue lives on the ticket, discovered in Load Ticket Context. Only if the caller explicitly adds a question/instruction → store as `<question>` to override/refine.

### Load Ticket Context

- If `<ticket-url>` defined: load ticket context via the issue tracker tool (see `/docs/issue-trackers.md` in the project root) with `source: <ticket-url>`, `comments: true` → `<ticket-context>`
  - Read all attachments with `relativePath` via `read` before triaging (images → describe; documents → extract key info → `<attachment-insights>`). Note gaps if inaccessible.
- Otherwise: treat relevant request/conversation as `<ticket-context>`
- If `<ticket-url>` missing or `<ticket-context>` cannot be loaded → STOP

### Determine Mode

Read the ticket body and every comment in order. Classify:

- **ask** — ticket contains a question needing an answer. The question may come from the client opening message or an internal note/comment from a teammate. The question is the most recent unanswered question. An explicit `<question>` from arguments takes precedence.
- **debug** — ticket reports a bug, unexpected behavior, or error needing root-cause investigation. Derive `<issue-description>`, `<expected-behavior>`, `<actual-behavior>`, `<reproduction-steps>`, `<error-messages>`, `<environment-context>`. Don't discard earlier comments.

If ambiguous after reading the full discussion, ask for clarification. If both apply, default to debug (root-cause investigation answers the question too).

Store the mode as `<triage-mode>`.

### Gather Evidence

**ask mode:**
- Repository context needed: gather a distilled brief — entry points, callers, conventions, file:line citations — using `grep`/`find_file_by_name`/code navigation tool (see `/docs/code-navigation.md` in the project root). Pass `<question>`, relevant entities, module path. Store as `<repo-context>`.
- Library/framework/SDK docs needed: gather a distilled brief on specific API/version questions using doc lookup (see `/docs/doc-lookup.md` in the project root). Store as `<doc-context>`.

**debug mode:**
- Run the context-gathering workflow: grep/find_file_by_name/code navigation tool (see `/docs/code-navigation.md` in the project root) for symbols, database tools (see `/docs/database-tools.md` in the project root) for data, doc lookup (see `/docs/doc-lookup.md` in the project root) for framework docs. Trace the code path from user action to failure; identify where actual behavior diverges from expected. Treat the live database as the source of truth for data. Verify current schema, row counts, sample rows, and actual values directly; do not rely on seeders, migrations, or operations to establish current state. Store findings as `<investigation-findings>`.

### Draft Findings

Synthesize evidence into a response. Mode determines the shape.

**ask mode:** Answer `<question>` using ticket discussion plus `<repo-context>` / `<doc-context>` when gathered. Store as `<ticket-findings>`.

**debug mode:** Build a root-cause note. Reproduction + ranked hypotheses separates root-cause finding from symptom-guessing.

Run the debug investigation steps below:

#### Ranked Hypotheses (debug)

Generate 3–5 ranked hypotheses before testing. Single-hypothesis generation anchors on the first plausible idea. Each must be falsifiable: state the prediction. Format: "If `<X>` is the cause, then `<changing Y>` will make the bug disappear." If no prediction can be stated, discard it — that is a vibe, not a hypothesis. Show the ranked list to the user before concluding. Don't block if AFK.

#### Debug Instrumentation (debug)

When adding temporary logs/queries to distinguish hypotheses:
- Tag every debug log with a unique prefix, e.g., `[DEBUG-a4f2]` (cleanup becomes a single grep)
- Change one variable at a time (each probe maps to a specific prediction from the ranked list)
- Never "log everything and grep" — targeted logs at boundaries that distinguish hypotheses

#### Analyze Root Cause (debug)

Synthesize codebase inspection, `<database-findings>`, `<error-findings>`, `<doc-context>`, `<skill-context>` to determine:
- `<root-cause>` — fundamental reason the issue occurs
- `<affected-components>` — which parts are involved
- `<impact-scope>` — how widespread (single user, single company, all users)
- `<reproduction-conditions>` — specific conditions triggering the issue
- `<potential-fixes>` — possible approaches (without implementing)

#### Completion Criterion — Root Cause Identified (debug)

Phase done when ALL hold:
- [ ] **Reproduced** — bug confirmed against user's exact symptom (not nearby failure)
- [ ] **Ranked** — 3–5 hypotheses generated, each falsifiable with stated prediction
- [ ] **Tested** — winning hypothesis tested and prediction held; alternatives falsified/ranked below
- [ ] **Scoped** — `<root-cause>`, `<affected-components>`, `<impact-scope>`, `<reproduction-conditions>` all stated with evidence
- [ ] **Gaps named** — if root cause can't be fully determined, state what's known and what's needed

Ground analysis in actual findings. Avoid speculation.

#### Shape Investigation Findings (debug)

Map `<investigation-findings>` to the published note. Carry every field forward; do not drop CONFIDENCE, ALTERNATIVE_HYPOTHESES, or GAPS.

- `<note-title>` — clear title summarizing the investigation (derive from ROOT_CAUSE)
- `<issue-summary>` — brief description of what was investigated (from the issue context)
- `<root-cause-analysis>` — detailed explanation (from ROOT_CAUSE + EVIDENCE)
- `<confidence>` — confidence level + justification (from CONFIDENCE)
- `<evidence>` — code snippets, query results, logs (from EVIDENCE + CODEBASE_FINDINGS + DATABASE_FINDINGS + ERROR_FINDINGS)
- `<affected-areas>` — components/modules/features affected (from AFFECTED_COMPONENTS)
- `<alternative-hypotheses>` — ranked alternatives and why they were ruled out (from ALTERNATIVE_HYPOTHESES)
- `<recommendations>` — suggested next steps/fix approaches (from POTENTIAL_FIXES / GAPS)
- `<gaps>` — what's unknown and what's needed to close it (from GAPS)

Include file paths, line numbers. Store as `<ticket-findings>`.

### Ticket Framing

Applies to both modes. See `/docs/issue-trackers.md` in the project root for provider-specific framing rules (private notes vs public comments).

- Frame findings based on the ticket provider's visibility rules. When the audience is internal team only, use technical language, cite code paths, state what was found and what it means. Never write as if speaking to the client (no "Hi", no "Thanks for reaching out", no second-person address to the customer).
- Do NOT modify, create, or delete files; do NOT run git commit or push.

### Sync

CRITICAL: Always sync findings back to the ticket. The spec phase reads prior comments via the ticket load tool; skipping sync breaks the pipeline.
- Sync findings via the issue tracker sync tool (see `/docs/issue-trackers.md` in the project root). Pass `refUrl` = `<ticket-url>` and `comments` = array with single markdown string. Debug mode: combine `<note-title>` (as top `##` heading) and structured findings. Ask mode: pass `<ticket-findings>` directly.
- Follow provider-specific sync rules from `/docs/issue-trackers.md` in the project root (e.g., omit `title` when updating existing cards or posting private notes).
- If sync fails, report the error and provide findings in chat.

## Notes

- The ticket load tool downloads attachments to `storage/app/mcp/{source}`. Read before triaging.
- Private note: technical and concise. Cite by file path + line range; don't paste full code blocks.
- Final output: ticket URL (+ note title for debug mode). Don't paste the full findings back; they are on the ticket.

## Common Rationalizations

| Excuse | Reality |
|---|---|
| "I'll just fix it directly" | Fixing without root cause guarantees recurrence. |
| "The bug is obvious" | Obvious bugs are usually symptoms, not causes. |
| "Debugging takes too long" | Systematic debugging is faster than trial-and-error. |

## Red Flags
**Never:**
- Propose a fix before root cause is identified (debug).
- Conclude root cause without a reproduction confirmed against the user's exact symptom (debug).
- Generate a single hypothesis — anchor on the first plausible idea and you miss the cause (debug).
- State a hypothesis that has no falsifiable prediction — vibes aren't hypotheses (debug).
- Skip the investigation steps (debug).
- Make code changes — this skill is READ-ONLY.
- Guess at root cause without evidence (debug).

**Always:**
- Stop and say so explicitly when you cannot reproduce. List what you tried, then ask (debug).
- Generate 3–5 ranked hypotheses before concluding (debug).
- Show the ranked list to the user before concluding (don't block if AFK) (debug).
- Tag debug logs with a unique `[DEBUG-xxxx]` prefix (debug).
- Publish findings per the ticket provider's visibility rules.
- Ground analysis in actual findings; avoid speculation.
- Cite file paths and line numbers in the note.
- Ask "what would have prevented this?" and flag architectural gaps (debug).

