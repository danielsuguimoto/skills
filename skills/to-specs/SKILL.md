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

**Effort scales with the task.** A simple, localized bug gets a simple spec: context limited to the code the fix touches, no brainstorming, few items. Do not manufacture complexity — no invented subsystems, speculative hardening, new dependencies, or restyled code for a fix that fits existing patterns. Spec size matches change size: a one-line fix produces a short spec and a single patch item.

**Loop with the user, never with yourself.** All iteration lives in the grilling phase (step 4) and the user-review gate (step 9): no internal review/refactor/self-critique cycles, no drafts or partial specs to the user.

**Ask via the environment's interactive question tool when one exists.**

<spec-template>

## Destination — the spec, decision, or change this effort delivers. One or two lines.
## Problem Statement — the user-facing problem or opportunity that drives the spec.
## Solution — the intended outcome from the user's perspective. Record each locked decision from grilling or repo inspection on one line with its source.
## Notes — other context, dependencies, or decisions worth keeping. The description renders only the sections above — Implementation Items and Validation Items sync as checklists and a patches comment (see step 10).

</spec-template>

1. **Interpret Arguments**: Ticket reference/URL → `<ticket-url>`. Any other input → `<request>`. Extra focus areas or constraints → `<additional-context>`. No arguments → use the ticket list tool to discover unresolved assigned tickets first; else derive `<request>` from the conversation.

2. **Load Planning Context**: If `<ticket-url>` exists, load ticket context via the issue tracker tool (see `<project-root>/docs/issue-trackers.md`), store as `<planning-context>`. Read each attachment — downloaded to `storage/app/mcp/{source}` — with `relativePath` via `read`. Describe images, extract key requirements from documents into `<attachment-insights>`, note inaccessible attachments. If `<ticket-url>` absent, treat the request or conversation as `<planning-context>`. Empty or missing → stop.

3. **Interpret Planning Context**: Derive from the full ticket (raw text + comments): `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, `<open-questions>` (unresolved only), `<spec-ticket-url>` (a linked spec-destination ticket on the issue tracker — e.g., a Trello card URL in a Crisp conversation's comments or links; empty when none). Keep earlier comments that define constraints, business rules, implementation decisions, migration rules, naming, sequencing, scoping. Multiple independent subsystems → flag immediately, help the user identify the pieces, relationships, build order; plan the first subsystem through the normal flow. Big problems: name the destination first, keep all output on one ticket; split into separate tickets only when the user asks.

4. **Clarify Requirements with User (MANDATORY)**: Run a clarification interview before shaping by default. Only place iteration happens before shaping — loop with the user until shared understanding. During the interview, ask the next question; do not restate previous answers. Skipping is the exception; justify it in the spec output.

**Scope of grilling — code-answerable questions are out of bounds.** Before asking anything, try to resolve it from repo, schema, docs, existing tickets, `<project-standards>`. Questions about current behavior, existing patterns, callers, file locations, symbol names, data flow, schema shape, "how does X work today" → answered by inspection, never asked. Record from inspection, move on. Ask only about critical decisions and genuine uncertainty the codebase cannot settle: scope boundaries and out-of-scope, business rules and product behavior choices, trade-offs between valid approaches, naming/placement when no existing pattern guides it, conflicts between the request and existing invariants. Each question carries a recommended answer grounded in what the code does today, citing `file:line` where relevant.

Skip gate (all true): single atomic edit or small localized fix with no design decisions; `<proposed-technical-direction>` concrete and unambiguous; no earlier comments conflict; no open questions about scope, naming, placement, ordering, behavior; repo inspected and direction matches existing patterns; the request defines scope and expected behavior unambiguously on its own (defect reports with expected-vs-actual qualify) or the user explicitly confirmed in the current conversation. Skip reason (when used): `Skip reason: <all 6 conditions met because ...>` — skipping without justification is a process violation. Unclear request or any gate false/uncertain → invoke a clarification interview. If an accessible grilling or interview skill exists, load it and follow its discipline. Pass planning context, `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, `<open-questions>`, relevant module path(s). Store resolved decisions as `<clarified-requirements>`. Continue only after shared understanding. Reflect `<clarified-requirements>` in `<spec-description>`, `<requirement-items>`, or `<validation-items>`. Do not proceed to step 5 until all user inputs captured.

5. **Gather Context** (project standards, repo, docs, skills):
- **Project standards**: Read relevant `<project-root>/docs/` specs (project root and module-specific). Check the active memory provider for code standards, architecture, tech stack. Store as `<project-standards>`.
- **Repo**: Entry points, existing patterns, callers, schema facts, `file:line` citations, validation of `<proposed-technical-direction>`, and a **surface inventory** of each layer the change touches (DB, model, policy, service, controller, API, UI, tests, config, docs, ops). For each layer: patch, new file, or no change, with concrete file(s) and symbol(s). Store as `<repo-context>`. `grep`/`find_file_by_name` for names, code navigation tool (see `<project-root>/docs/code-navigation.md`) for symbols/references, database tools (see `<project-root>/docs/database-tools.md`) for schema. Read one or two key files. Confirm current behavior and patterns. Note gaps; avoid false certainty. Scale the scan to the change — a localized bug maps the failing path and its immediate callers, not a full-layer inventory.
- **Docs**: API signatures, version-specific behavior, gotchas for libraries, frameworks, Laravel features, `<proposed-technical-direction>`, API/best-practice questions. Store as `<doc-context>`. Use doc lookup per `<project-root>/docs/doc-lookup.md`.
- **Skills**: Planning touches specific domains (testing, frontend, Livewire) → invoke the relevant skills. Store as `<skill-context>`.

6. **Brainstorm Technical Approaches**: After inspecting the repo, brainstorm for complex tasks. Complex = touches multiple subsystems, multiple valid high-level approaches, lacks concrete `<proposed-technical-direction>`, or user flags it. Complex tasks: ask remaining clarifying questions one at a time — same grilling rule as step 4 (approach trade-offs, scope, business rules qualify); propose 2-3 approaches grounded in `<repo-context>` with trade-offs; lead with a recommendation; ask the user to choose one. Record as `<technical-approach-decision>`, get explicit alignment before shaping. A defect whose fix follows existing patterns is non-complex by default. Non-complex tasks: skip when `<proposed-technical-direction>` is concrete and unambiguous and the fix follows directly from inspection — user confirmation is required only when direction is absent or ambiguous.

7. **Build Action Inventory**: Before writing requirement items, list every concrete code change needed to deliver the destination. Use the surface inventory from `<repo-context>`. For small localized changes, enumerate only the affected surfaces and collapse the rest to a single `no change` line. For each surface (DB, model, rules, services, API, UI, infra, tests, docs, ops), decide `no change` or a concrete action (`create`, `modify`, `delete`). Record `surface`, `action`, `target`, `depends-on`, `patch-ready` (`true` when the exact code change can be written from `<repo-context>`). Convert each `patch-ready` action into a requirement item in step 8. Every required action becomes a requirement item.

8. **Shape the Spec** (internal — do not present to the user): Write the spec using the template. Turn `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, `<clarified-requirements>`, `<technical-approach-decision>`, repo findings, `<project-standards>`, `<doc-context>`, `<skill-context>` into:
- `<spec-title>`: short, useful title.
- `<spec-description>`: the rendered spec template (Destination, Problem Statement, Solution, Notes only — NOT implementation/validation items). Must include: (a) destination and why it fixes scope, (b) chosen technical approach and why, (c) key user preferences and constraints, (d) accepted trade-offs. Specific to this spec and user.
- `<requirement-items>`: precise patch descriptions, one per spec step. Map each action from step 7 and each user preference from `<clarified-requirements>` to one or more requirement items.
- `<validation-items>`: validation checklist aligned with project testing conventions. Adhere to `<project-standards>`, `<doc-context>`, `<skill-context>`. Preserve valid technical details; improve incomplete ones when repo inspection gives better direction. No placeholder labels.

**Requirement Item Format**: Each `<requirement-item>` is a self-contained block an implementation agent can apply as a patch. No item describes an outcome without naming where the code changes. Required fields: `id` (stable spec-step-id like `S1` or short slug), `file` (repo-root-relative path(s); locator rule if multiple candidates), `symbol` (function/class/method/component; fully-qualified when ambiguous), `location` (exact insertion or modification point), `instruction` (one short imperative sentence: `add`/`remove`/`modify`/`replace`), `patch` (minimal code snippet or unified diff; required when target known; keep to the delta).

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

Rules: One item = one single-purpose patch. Split compound changes into separate items. Broader changes → sub-items (`S1a`, `S1b`). Verify paths and symbols against `<repo-context>`; no guesses. Cannot confirm a target → flag it in the item, let the user catch it at step 9 review. No exploratory steps ("investigate X", "consider Y", "ensure X", "update X as needed"). No alternative designs or re-evaluation after the user agrees on direction. Items describe code changes only. Runtime operations belong to the deploy process, not the spec: migrations, seeders, cache clears, queue restarts, and other user/ops actions are excluded even when the change requires them. The spec lists only what is included: exclusions decided during grilling shape the scope silently and appear nowhere in the spec — no "out of scope" lines, no mentions of excluded items.

9. **User Review** (the only spec presentation): Present the full shaped spec exactly once — `<spec-title>`, `<spec-description>`, `<requirement-items>`, `<validation-items>`. No brief or summary version first then the full version; one presentation only. In the same gate, reflect the user's inputs back: list 3-5 key decisions or constraints and where they appear (cite requirement/validation items); confirm the spec covers all concrete actions; then ask:
> Spec ready: `<spec-title>`. Implementation: N items. Validation: N items. Sync to the ticket system, or revise the spec?

The only two outcomes of this gate: sync or revise — never offer to apply, implement, or execute the spec; user asks to implement → decline, point to the implementation workflow. Wait for the user's response; sync only after approval.

10. **Sync Ticket**: Sync the spec to the ticket system via the issue tracker sync tool (see `<project-root>/docs/issue-trackers.md`). Final output: the ticket URL.
- `title`: `<spec-title>`, `description`: `<spec-description>` (Destination/Problem/Solution/Notes only — no implementation/validation items, no patch code).
- `checklists`: two non-empty sections as JSON array matching `{name, items: [{name, completed}]}`:
  - `{"name": "Implementation", "items": [{"name": "<requirement-item-encoded>", "completed": false}, ...]}`
  - `{"name": "Validation", "items": [{"name": "<validation-item>", "completed": false}, ...]}`
  - `<requirement-item-encoded>`: `S1: <title> - <file> @ <symbol>::<location> - <instruction>` (no `patch` code — checklists are short tracking items).
  - Before calling, verify the `checklists` schema against the sync tool's source.
- `patches comment`: post the full structured `<requirement-items>` blocks (with `patch` code) as a comment on the ticket via the issue tracker comment operation (see `<project-root>/docs/issue-trackers.md`). Keeps implementer code out of the card body, which overflows ticket systems with card-body limits (e.g. Trello). Skip only when the tracker has no comment support; then fall back to attaching the blocks as a file.
- `refUrl` — resolve the destination deterministically, never by asking. Load the resolved ticket's state before deciding; replace or delete an existing ticket only when the user asks:
  - `<spec-ticket-url>` set and the ticket is open → update it.
  - `<ticket-url>` points at a ticket on the spec tracker itself and it is open → update it.
  - Resolved ticket flagged completed/done → never reopen or reuse it — create instead.
  - No open spec ticket → omit `refUrl`, create the ticket (asking whether to create is a process violation), post its URL back to the source ticket when it lives on a different tracker and supports comments/links, output the new URL.

