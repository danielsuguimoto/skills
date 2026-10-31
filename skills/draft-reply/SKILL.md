---
name: draft-reply
description: "Draft a reply to a ticket and post it as a private note for review. READ-ONLY — no code changes, no public reply."
---

## Required `<project-root>/docs` reads

Shell-read before drafting the reply; may be gitignored — missing → note gap, never invent: `<project-root>/docs/code-navigation.md`, `<project-root>/docs/database-tools.md`, `<project-root>/docs/doc-lookup.md`, `<project-root>/docs/issue-trackers.md`

Load and analyze the ticket, gather context if needed, draft a reply, post it as a private note for human review. READ-ONLY — the note is a draft the human sends; it promises no work (changes, commits, PRs) unless `<additional-context>` explicitly asks.

## Additional Context

Treat ticket systems generically. `<additional-context>` shapes tone, audience, depth — default audience is the customer/requester. One draft per ticket.

## The Discipline

**The draft is for a human, not the customer.** Ready to send verbatim; the human decides whether and how to deliver it.

### Interpret Arguments

Ticket reference/URL → `<ticket-url>`; extra focus/constraints/audience hints → `<additional-context>`; none → derive from conversation. No `<question>` expected in arguments — it lives on the ticket; an explicit one → store as `<question>` to scope the reply.

### Load Ticket Context

- `<ticket-url>` → issue tracker tool (`issue-trackers.md`) with `source: <ticket-url>`, `comments: true` → `<ticket-context>`; attachments per spec → `<attachment-insights>`.
- Otherwise → the request/conversation is `<ticket-context>`.
- Missing or unloadable → STOP.

### Analyze the Ticket

Read body + every comment in order. Derive:

- `<reply-target>` — most recent unanswered question or open request the reply must address; multiple open threads → pick most recent, note others.
- `<requester-profile>` — author, role, technical level (infer from tone/content; don't assume).
- `<ticket-facts>` — stated facts, constraints, prior answers, what's resolved.
- `<open-gaps>` — still unclear/unaddressed items the reply must cover.

Ambiguous about the reply target → ask the user before drafting. Never guess `<reply-target>`.

### Gather Codebase Context (only if needed)

Skip when the reply is purely procedural/status/non-technical and the ticket already answers it. Otherwise:

- **Code**: distilled brief (entry points, callers, conventions, `file:line` citations) via `grep`/`find_file_by_name`/code navigation (`code-navigation.md`); pass `<reply-target>`, relevant entities, module path → `<repo-context>`.
- **Data**: reply depends on persisted state (counts, status, samples) → live DB via database tools (`database-tools.md`) — ground truth.
- **Docs**: framework/library/API/version questions → doc lookup (`doc-lookup.md`) → `<doc-context>`.

Store as `<supporting-context>`; cite `file:line`/query results inline where they back a claim — never dump raw output into the draft.

### Draft the Reply

Write `<draft-reply>` as a message the human can send with minimal edits:

- **Audience match.** Tone/depth per `<requester-profile>` + `<additional-context>`. Customer-facing default: greet, plain language, no jargon, no `file:line` in the body. Internal reviewer: technical, cite `file:line`, no client tone.
- **Answer first.** Lead with the answer to `<reply-target>`; background/caveats after.
- **Human prose.** Write `<draft-reply>` per the `writing-for-humans` skill — it is text for a human reader: no staging, stock vocabulary, formatting tics, or chat residue, and keep the requester's own vocabulary where the draft mirrors their words.
- **Ground every claim.** Technical assertions trace to `<ticket-facts>`, `<supporting-context>`, or `<doc-context>`; ungrounded → `[NEEDS-REVIEW: <what's uncertain>]`, not stated as fact.
- **No invented details.** No fabricated dates, IDs, statuses, quotes — say what's missing, flag for the human.
- **Length matches the question.** One-line answer for a one-line question; structured for multi-part. No padding.

### Shape the Note

The private note is `<draft-reply>` and nothing else — ready to send verbatim. No headings, supporting sections, or analysis. `[NEEDS-REVIEW]` flags stay inline in the body for the human to resolve before sending.

### Sync

Post via the issue tracker sync tool — convention per `issue-trackers.md` (`refUrl` = `<ticket-url>`; private note per provider, `DRAFT` prefix fallback). Fails → report error, provide draft in chat.

## Notes

- Final output: ticket URL of the posted note — one line confirming, never the full draft pasted back.
- Revision after posting → edit the note in place per provider rules; never post a second draft.
