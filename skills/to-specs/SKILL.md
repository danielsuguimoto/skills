---
name: to-specs
description: "Turn a request or ticket into a change and test spec, standalone or after triage, and sync it to the ticket system; leave application code unchanged."
---

## Required `<project-root>/docs` reads

Shell-read before turning the request into a spec (reuse session reads): `<project-root>/docs/code-navigation.md`, `<project-root>/docs/issue-trackers.md`; only when the spec touches them — `<project-root>/docs/database-tools.md` (persisted state/schema), `<project-root>/docs/doc-lookup.md` (library/API uncertainty). If a spec is missing, report the gap and use a documented fallback.

Treat ticket systems generically.

**Effort scales with the task.** Gather only evidence needed to define the requested outcome and a viable approach. A localized fix usually needs one implementation item and one validation item; add more only when distinct outcomes or checks warrant them.

**Surgical scope.** Each item traces to the request, ticket, or a locked user decision. Unrelated findings belong in Notes, not implementation items.

**Simplest solution wins.** Prefer existing surfaces and the least complex change that satisfies the request and explicit constraints.

**Spec boundary — implementation and automated tests only.** Every spec item is a code change or an executable automated check. Manual/exploratory validation, QA walkthroughs, and environment/ops checks belong to review and deploy, outside the spec — never an item, never an interview question.

**Loop with the user, never with yourself.** All iteration lives in the grilling phase (step 5) and the user-review gate (step 9): no internal review/refactor/self-critique cycles, no drafts or partial specs to the user. Ask every user-facing question via the interactive question tool when one exists — a bare text question ends the turn looking finished instead of awaiting input.

<spec-template>

## Destination — the outcome this effort delivers.
## Problem Statement — the user-facing problem or opportunity.
## Solution — the simplest viable approach and explicit constraints; distinguish required decisions from suggested mechanics.
## Notes — include only material context or dependencies. Implementation and Validation items sync as checklists (step 10).

</spec-template>

1. **Interpret Arguments**: Explicit ticket reference/URL → `<ticket-url>`; any other input → `<request>`; extra focus areas or constraints → `<additional-context>`. A written `<request>` is authoritative — skip session inference. Without a ticket or `<request>`, use the unambiguous ticket targeted by the session's current work; ambiguous → ask. Without a ticket or arguments → use the ticket list tool to discover unresolved assigned tickets first, then fall back to `<request>` from the conversation.

2. **Load Planning Context**: `<ticket-url>` exists → reuse a complete, current in-session copy — full comment thread, this session's synced notes appended — or load it via the issue tracker tool (`issue-trackers.md`), store as `<planning-context>`; attachments per spec → `<attachment-insights>`. No `<ticket-url>` → the request/conversation is `<planning-context>`. Empty or missing → stop.

3. **Interpret Planning Context**: Derive from the full ticket (raw text + comments): `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, `<open-questions>` (unresolved only), `<spec-ticket-url>` (a linked spec-destination ticket URL in the source ticket's body/comments/links; empty when none). Keep earlier comments defining constraints, business rules, implementation decisions, migration rules, naming, sequencing, scoping. Anchor `<source-request>` — the requester's own statement (ticket body + requester comments), distinct from internal discussion and triage findings: those inform the spec as evidence but never replace what the requester asked. Multiple independent subsystems → flag immediately; help the user identify pieces, relationships, build order; plan the first subsystem through the normal flow. Big problems: name the destination first, keep output on one ticket; split only when the user asks.

4. **Gather Context** (standards, repo, docs, skills) — runs before any user question, so the interview never asks what the codebase already answers:
- **Sync**: fetch per `git-hosts.md`; the checked-out branch behind → fast-forward a clean tree, dirty/diverged → flag staleness — spec items embed `file`/`location` targets that drift on stale code.
- **Project standards**: relevant `<project-root>/docs/` specs and active memory only when needed to decide the change → `<project-standards>`.
- **Repo**: inspect the affected path and relevant callers; confirm current behavior and validate `<proposed-technical-direction>` → `<repo-context>`. Verify reuse leads from triage against current code. If reuse is unresolved, check the nearest analogous flow and relevant shared code for the same behavior. Use database tools when schema or live data matters. Stop when the outcome and viable approach are grounded.
- **Docs**: resolve material library/API uncertainty per `<project-root>/docs/doc-lookup.md` → `<doc-context>`.
- **Skills**: planning touches specific domains (testing, frontend, Livewire) → invoke the relevant skills → `<skill-context>`.

5. **Clarify Requirements with User**: Ask only when a material product, scope, or design decision remains unresolved after step 4. Otherwise proceed directly; no skip justification in the spec. Ask the next question; never restate previous answers.

**Grilling scope — code-answerable questions are out of bounds; step 4 already answered them.** Resolve candidate questions against the gathered context first. Ask only about material scope, behavior, trade-offs, or conflicts the codebase and request cannot settle. Each question names what was checked, explains each option in one line, and flags the recommendation (`(recommended)` on its label), grounded in `<repo-context>`. Validation approach is not a scope question — the spec boundary already fixes it (automated checks per project conventions).

When a decision needs the user, use the grilling/interview skill if available, passing the relevant planning and repo context. Store resolved decisions as `<clarified-requirements>`; reflect them in the spec. Do not proceed until material decisions are resolved.

6. **Choose Technical Approach**: wire the existing implementation into the affected module when its contract and side effects fit. If direct reuse falls short, prefer the smallest compatible change to existing code; propose a new service, class, layer, or file only when existing code cannot safely cover a required behavior. Compare alternatives only when a material trade-off affects the outcome or an explicit constraint. Ask the user to choose only when that trade-off requires their decision → `<technical-approach-decision>`.

7. **Build Action Inventory**: identify the fewest request-bounded changes needed to deliver the destination, using affected targets from `<repo-context>`. Group closely related edits by deliverable; omit unchanged surfaces and speculative hardening.

8. **Shape the Spec**: turn `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, `<clarified-requirements>`, `<technical-approach-decision>`, repo findings, `<project-standards>`, `<doc-context>`, `<skill-context>` into the spec below. Write per `writing-for-agents`: bullets over paragraphs, one fact or decision per line, `file:line` citations where useful, no motivational prose.
- `<spec-title>`: short, useful title.
- `<spec-description>`: concise template; restate `<source-request>` in the requester's terms and name the intended outcome, approach, and material constraints. In Solution, cite the existing implementation and wiring point (`file:line`). If proposing new structure, cite the closest candidate or say where you looked, then name the gap that requires it. Omit empty Notes. Suggested mechanics guide implementation; explicit requirements and user decisions bind it.
- `<requirement-items>`: short, outcome-oriented items for the smallest set of deliverables. Use stable IDs (`S1`, `S2`, ...); group related edits that serve one outcome instead of one item per patch or surface. Name verified files/symbols when useful, not guessed locations. Include a minimal patch only when it clarifies a non-obvious decision.
- `<validation-items>`: the fewest meaningful automated checks or assertions that demonstrate the requested behavior, aligned with project conventions. Reuse existing checks where suitable; do not prescribe new tests for trivial changes by default. If none is suitable, flag the validation gap for review rather than inventing a check. Manual verification, exploratory QA, and environment/ops checks belong to review or deploy.

Trace each item to the request or an explicit decision; cut unneeded items. Flag an unconfirmed target or material uncertainty for review. Code changes only — runtime operations belong to deploy. The spec lists only what is included.

9. **User Review** (the only spec presentation): present the full, concise spec exactly once — `<spec-title>`, `<spec-description>`, `<requirement-items>`, `<validation-items>` — and flag any material uncertainty. Check coverage and traceability internally; remove unneeded items before presenting. Ask via the interactive question tool — header `Spec Ready`, options `Sync to Ticket` / `Revise Spec`:
> Spec ready: `<spec-title>`. Sync the spec, any patches, and any required source-ticket link, or revise the spec?

Only two outcomes: sync or revise — never offer to apply/implement/execute; user asks to implement → decline, point to the implementation workflow. `Revise Spec` → resolve the revision points (step 5), reshape, re-enter this gate; `Sync to Ticket` → step 10. Sync only on that answer. That answer authorizes this spec's Step 10 batch: create or update the resolved spec ticket, add or supersede any patches comment or attachment, and link it from the source ticket when applicable. Do not ask again for those writes. A material change to the spec or destination beyond the disclosed sync scope requires renewed approval. Honor tool-enforced permissions.

10. **Sync Ticket**: sync via the issue tracker sync tool (`issue-trackers.md`); the synced spec joins the in-session ticket context for downstream reuse. Final output: the ticket URL.
- `title`: `<spec-title>`; `description`: `<spec-description>` (Destination/Problem/Solution and optional Notes — no items, no patch code).
- `checklists`: JSON array matching `{name, items: [{name, completed}]}`; omit Validation when no suitable automated check exists:
  - `{"name": "Implementation", "items": [{"name": "<requirement-item-encoded>", "completed": false}, ...]}`
  - `{"name": "Validation", "items": [{"name": "<validation-item>", "completed": false}, ...]}`
  - `<requirement-item-encoded>`: `S1: <outcome> - <verified targets when useful>` (no `patch` code — checklists are short tracking items).
  - Verify the `checklists` schema against the sync tool's source before calling.
- `patches comment`: only when a patch clarifies a non-obvious decision, post it with its item ID as a ticket comment; if comments are unavailable, attach it as a file. When updating a ticket with older patches, identify them as superseded in the new comment or attachment, including when the revision has no patches.
- `refUrl` — resolve deterministically, never by asking; load the resolved ticket's state first; replace/delete an existing ticket only when the user asks:
  - `<spec-ticket-url>` set and open → update it.
  - `<ticket-url>` points at a ticket on the spec tracker itself and open → update it.
  - Resolved ticket flagged completed/done → never reopen/reuse — create instead.
  - No open spec ticket → omit `refUrl`, create the ticket (asking whether to create is a process violation), post its URL back to the source ticket when it lives on a different tracker and supports comments/links, output the new URL.
