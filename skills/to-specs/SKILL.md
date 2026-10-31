---
name: to-specs
description: Use when a request or ticket needs to be turned into a spec and synced to the ticket system. Read-only (no code changes or implementation).
---

## Required `<project-root>/docs` reads

Read via shell before turning the request into a spec — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/code-navigation.md`
- `<project-root>/docs/database-tools.md`
- `<project-root>/docs/doc-lookup.md`
- `<project-root>/docs/issue-trackers.md`

Treat ticket systems generically.

**Effort scales with the task.** A simple localized bug gets a simple spec: context limited to the code the fix touches, no brainstorming, few items. No manufactured complexity — no invented subsystems, speculative hardening, new dependencies, or restyled code for a fix that fits existing patterns. Spec size matches change size: a one-line fix → a short spec + single patch item.

**Loop with the user, never with yourself.** All iteration lives in the grilling phase (step 4) and the user-review gate (step 9): no internal review/refactor/self-critique cycles, no drafts or partial specs to the user.

**Ask via the interactive question tool when one exists.**

<spec-template>

## Destination — the spec, decision, or change this effort delivers. One or two lines.
## Problem Statement — the user-facing problem or opportunity driving the spec.
## Solution — the intended outcome from the user's perspective. Each locked decision from grilling or repo inspection on one line with its source.
## Notes — other context, dependencies, or decisions worth keeping. The description renders only the sections above — Implementation and Validation Items sync as checklists and a patches comment (step 10).

</spec-template>

1. **Interpret Arguments**: Ticket reference/URL → `<ticket-url>`; any other input → `<request>`; extra focus areas or constraints → `<additional-context>`. No arguments → use the ticket list tool to discover unresolved assigned tickets first; else derive `<request>` from the conversation.

2. **Load Planning Context**: `<ticket-url>` exists → load ticket context via the issue tracker tool (`<project-root>/docs/issue-trackers.md`), store as `<planning-context>`. Read each attachment — downloaded to `storage/app/mcp/{source}` — with `relativePath` via `read`. Describe images, extract key requirements into `<attachment-insights>`, note inaccessible attachments. No `<ticket-url>` → the request/conversation is `<planning-context>`. Empty or missing → stop.

3. **Interpret Planning Context**: Derive from the full ticket (raw text + comments): `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, `<open-questions>` (unresolved only), `<spec-ticket-url>` (a linked spec-destination ticket — e.g., a Trello card URL in a Crisp conversation's comments/links; empty when none). Keep earlier comments defining constraints, business rules, implementation decisions, migration rules, naming, sequencing, scoping. Multiple independent subsystems → flag immediately; help the user identify pieces, relationships, build order; plan the first subsystem through the normal flow. Big problems: name the destination first, keep output on one ticket; split only when the user asks.

4. **Clarify Requirements with User (MANDATORY)**: Run a clarification interview before shaping by default — the only iteration before shaping; loop until shared understanding. Ask the next question; never restate previous answers. Skipping is the exception; justify it in the spec output.

**Grilling scope — code-answerable questions are out of bounds.** Resolve from repo, schema, docs, existing tickets, `<project-standards>` before asking. Questions about current behavior, existing patterns, callers, file locations, symbol names, data flow, schema shape, "how does X work today" → answered by inspection, never asked. Ask only what the codebase cannot settle: scope boundaries and out-of-scope, business rules and product behavior choices, trade-offs between valid approaches, naming/placement with no guiding pattern, conflicts between the request and existing invariants. Each question carries a recommended answer grounded in current code, citing `file:line`.

Skip gate (ALL true): single atomic edit or small localized fix, no design decisions · `<proposed-technical-direction>` concrete and unambiguous · no conflicting earlier comments · no open questions about scope, naming, placement, ordering, behavior · repo inspected, direction matches existing patterns · the request defines scope and expected behavior unambiguously (defect reports with expected-vs-actual qualify) or the user explicitly confirmed in this conversation. Skip reason (when used): `Skip reason: <all 6 conditions met because ...>` — skipping without justification is a process violation. Any gate false/uncertain → run the interview; if a grilling/interview skill exists, load it and follow its discipline, passing planning context, `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, `<open-questions>`, relevant module paths. Store resolved decisions as `<clarified-requirements>`; reflect them in `<spec-description>`, `<requirement-items>`, `<validation-items>`. Do not proceed to step 5 until all user inputs are captured.

5. **Gather Context** (standards, repo, docs, skills):
- **Project standards**: relevant `<project-root>/docs/` specs (root and module-specific); active memory provider for standards, architecture, tech stack → `<project-standards>`.
- **Repo**: entry points, patterns, callers, schema facts, `file:line` citations, validation of `<proposed-technical-direction>`, and a **surface inventory** per layer the change touches (DB, model, policy, service, controller, API, UI, tests, config, docs, ops) — patch, new file, or no change, with concrete files/symbols → `<repo-context>`. `grep`/`find_file_by_name` for names, code navigation for symbols/references, database tools for schema. Read one or two key files; confirm current behavior; note gaps; avoid false certainty. Scale the scan to the change — a localized bug maps the failing path and immediate callers, not a full-layer inventory.
- **Docs**: API signatures, version-specific behavior, gotchas for libraries/frameworks/Laravel, `<proposed-technical-direction>` → `<doc-context>` (per `<project-root>/docs/doc-lookup.md`).
- **Skills**: planning touches specific domains (testing, frontend, Livewire) → invoke the relevant skills → `<skill-context>`.

6. **Brainstorm Technical Approaches**: complex tasks only (multiple subsystems, multiple valid approaches, no concrete `<proposed-technical-direction>`, or user-flagged). Ask remaining clarifying questions one at a time — same grilling rule as step 4; propose 2-3 approaches grounded in `<repo-context>` with trade-offs; lead with a recommendation; ask the user to choose → `<technical-approach-decision>`; explicit alignment before shaping. A defect whose fix follows existing patterns is non-complex by default — skip when direction is concrete/unambiguous; user confirmation required only when direction is absent or ambiguous.

7. **Build Action Inventory**: list every concrete code change needed to deliver the destination, from `<repo-context>`'s surface inventory. Small localized changes → enumerate only affected surfaces, collapse the rest to one `no change` line. Per surface (DB, model, rules, services, API, UI, infra, tests, docs, ops): `no change` or a concrete action (`create`/`modify`/`delete`). Record `surface`, `action`, `target`, `depends-on`, `patch-ready` (`true` when the exact change can be written from `<repo-context>`). Every required action becomes a requirement item in step 8.

8. **Shape the Spec** (internal — never present to the user): turn `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, `<clarified-requirements>`, `<technical-approach-decision>`, repo findings, `<project-standards>`, `<doc-context>`, `<skill-context>` into the spec below. All spec output is consumed by an implementation agent — write per `writing-for-agents`: bullets over paragraphs, one fact or decision per line, `file:line` citations, no motivational prose.
- `<spec-title>`: short, useful title.
- `<spec-description>`: rendered spec template (Destination, Problem Statement, Solution, Notes only — no implementation/validation items). One line per fact: destination and why it fixes scope · chosen approach and why · key user preferences and constraints · accepted trade-offs.
- `<requirement-items>`: precise patch descriptions, one per spec step — map each step-7 action and each `<clarified-requirements>` preference to ≥1 item.
- `<validation-items>`: checklist aligned with project testing conventions (`<project-standards>`, `<doc-context>`, `<skill-context>`). No placeholder labels.

**Requirement Item Format**: each `<requirement-item>` is a self-contained block an implementation agent applies as a patch — no outcome-only descriptions. Fields: `id` (stable spec-step-id like `S1` or short slug) · `file` (repo-root-relative path(s); locator rule if multiple candidates) · `symbol` (function/class/method/component, fully-qualified when ambiguous) · `location` (exact insertion/modification point) · `instruction` (one short imperative: `add`/`remove`/`modify`/`replace`) · `patch` (minimal snippet or unified diff; required when target known; keep to the delta).

```
### S1: <title>
- file: app/Path/To/File.php
- symbol: ClassName::methodName
- location: inside methodName(), before `return $model;`
- instruction: Throw `ValidationException` when `$model->isLocked()`.
- patch:
  ```php
  if ($model->isLocked()) {
      throw ValidationException::withMessages(['status' => 'Model is locked.']);
  }
  ```
```

Rules: one item = one single-purpose patch — split compound changes; broader changes → sub-items (`S1a`, `S1b`). Verify paths/symbols against `<repo-context>`; no guesses — unconfirmable target → flag it in the item for step-9 review. No exploratory steps ("investigate X", "consider Y", "ensure X"). No alternative designs after the user agrees on direction. Code changes only — runtime operations (migrations, seeders, cache clears, queue restarts, user/ops actions) belong to deploy, not the spec. The spec lists only what is included: exclusions decided during grilling appear nowhere — no "out of scope" lines.

9. **User Review** (the only spec presentation): present the full shaped spec exactly once — `<spec-title>`, `<spec-description>`, `<requirement-items>`, `<validation-items>`; no brief version first. In the same gate, reflect the user's inputs: list 3-5 key decisions/constraints and where they appear (cite items); confirm coverage of all concrete actions; then ask:
> Spec ready: `<spec-title>`. Implementation: N items. Validation: N items. Sync to the ticket system, or revise the spec?

Only two outcomes: sync or revise — never offer to apply/implement/execute; user asks to implement → decline, point to the implementation workflow. Wait for the response; sync only after approval.

10. **Sync Ticket**: sync via the issue tracker sync tool (`<project-root>/docs/issue-trackers.md`). Final output: the ticket URL.
- `title`: `<spec-title>`; `description`: `<spec-description>` (Destination/Problem/Solution/Notes only — no items, no patch code).
- `checklists`: two non-empty sections as JSON array matching `{name, items: [{name, completed}]}`:
  - `{"name": "Implementation", "items": [{"name": "<requirement-item-encoded>", "completed": false}, ...]}`
  - `{"name": "Validation", "items": [{"name": "<validation-item>", "completed": false}, ...]}`
  - `<requirement-item-encoded>`: `S1: <title> - <file> @ <symbol>::<location> - <instruction>` (no `patch` code — checklists are short tracking items).
  - Verify the `checklists` schema against the sync tool's source before calling.
- `patches comment`: post the full `<requirement-items>` blocks (with `patch` code) as a ticket comment — keeps implementer code out of the card body, which overflows systems with body limits (e.g. Trello). Skip only when the tracker lacks comments → attach the blocks as a file.
- `refUrl` — resolve deterministically, never by asking; load the resolved ticket's state first; replace/delete an existing ticket only when the user asks:
  - `<spec-ticket-url>` set and open → update it.
  - `<ticket-url>` points at a ticket on the spec tracker itself and open → update it.
  - Resolved ticket flagged completed/done → never reopen/reuse — create instead.
  - No open spec ticket → omit `refUrl`, create the ticket (asking whether to create is a process violation), post its URL back to the source ticket when it lives on a different tracker and supports comments/links, output the new URL.
