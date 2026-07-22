---
name: to-specs
description: Use when a request or ticket needs to be turned into a spec and synced to the ticket system. Read-only (no code changes or implementation).
---

## Required `<project-root>/docs` reads

Read these project-root spec files before turning the request into a spec (use shell `cat`/`ls` — they may be in `.gitignore`, invisible to built-in search). Missing file → fall back to native tools, note the gap; never invent contents.

- `<project-root>/docs/code-navigation.md`
- `<project-root>/docs/database-tools.md`
- `<project-root>/docs/doc-lookup.md`
- `<project-root>/docs/issue-trackers.md`

Turn a request or ticket into a spec, then sync to the ticket system. Read-only (no code changes or implementation). Clarify requirements with the user before shaping the spec. No assumptions, guesses, or inferred intent. Treat ticket systems generically. Use `<additional-context>` for constraints and focus areas. Update existing tickets; create replacements only when asked. For big problems, name the destination first and keep all output on one ticket. Split into multiple tickets only when the user explicitly asks.

**Loop with the user, never with yourself.** All iteration happens in the grilling phase (step 5) and the user-review gate (step 10) as back-and-forth with the user. Do not run internal review/refactor/re-review cycles, re-shape loops, or self-critique passes. Shape once from gathered context, present, revise only on user request.

<spec-template>

## Destination — the spec, decision, or change this effort delivers. One or two lines.
## Problem Statement — the user-facing problem or opportunity that drives the spec.
## Solution — the intended outcome from the user's perspective. Record each locked decision from grilling or repo inspection on one line with its source.
## Notes — other context, dependencies, or decisions worth keeping.

The description renders only the sections above. Implementation Items and Validation Items are NOT part of the description — they sync as checklists (short encoded form) and a patches comment (full structured blocks with `patch` code). See step 11.

</spec-template>

1. **Interpret Arguments**: Map a ticket reference/URL to `<ticket-url>`. Map any other input to `<request>`. Map extra focus areas or constraints to `<additional-context>`. Derive `<request>` from the conversation when the user provides no arguments.

2. **Load Planning Context**: If `<ticket-url>` exists, load ticket context via the issue tracker tool (see `<project-root>/docs/issue-trackers.md` in the project root) and store as `<planning-context>`. Read each attachment with `relativePath` via `read`. Describe images, extract key requirements from documents into `<attachment-insights>`, note inaccessible attachments. If `<ticket-url>` is absent, treat the request or conversation as `<planning-context>`. If empty or missing, stop.

3. **Interpret Planning Context**: Derive from the full ticket (raw text + comments): `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, and `<open-questions>` (only unresolved issues). Keep earlier comments that define constraints, business rules, implementation decisions, migration rules, naming, sequencing, or scoping. If the request describes multiple independent subsystems, flag it immediately. Help the user identify the independent pieces, their relationships, and the build order. Plan the first subsystem through the normal flow. Split into separate tickets only when the user explicitly asks.

4. **Gather Project Standards**: Read the relevant `<project-root>/docs/` specifications in the project root (project root and module-specific). Check the active memory provider for code standards, architecture, and tech stack. Store as `<project-standards>`.

5. **Clarify Requirements with User (MANDATORY)**: Run a clarification interview before shaping the spec by default. This is the only place iteration happens before shaping — loop with the user here until shared understanding. Skipping is the exception; justify it in the spec output. No assumptions, guesses, or inferred intent instead of user clarification.

Skip gate (all must be true): single atomic edit with no design decisions; `<proposed-technical-direction>` concrete and unambiguous; no earlier comments conflict; no open questions about scope, naming, placement, ordering, or behavior; repo inspected and direction matches existing patterns; user explicitly confirmed in the current conversation.

If the request is unclear or any gate is false or uncertain, invoke a clarification interview. Pass planning context, `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, `<open-questions>`, and relevant module path(s). Store resolved decisions as `<clarified-requirements>`. Continue only after shared understanding. Reflect `<clarified-requirements>` in `<spec-description>`, `<requirement-items>`, or `<validation-items>`. Do not proceed to step 6 until you capture all user inputs.

Skip reason (when used): `Skip reason: <all 6 conditions met because ...>`. Skipping without justification is a process violation.

6. **Gather Context** (repo, docs, skills):
- **Repo**: Gather entry points, existing patterns, callers, schema facts, `file:line` citations, validation of `<proposed-technical-direction>`, and a **surface inventory** of each layer the change touches (DB, model, policy, service, controller, API, UI, tests, config, docs, ops). For each layer state: patch, new file, or no change, with concrete file(s) and symbol(s). Store as `<repo-context>`. Use `grep`/`find_file_by_name` for names, code navigation tool (see `<project-root>/docs/code-navigation.md` in the project root) for symbols/references, database tools (see `<project-root>/docs/database-tools.md` in the project root) for schema. Read one or two key files. Confirm current behavior and patterns. Note gaps; avoid false certainty.
- **Docs**: Gather API signatures, version-specific behavior, gotchas for libraries, frameworks, Laravel features, `<proposed-technical-direction>`, API/best-practice questions. Store as `<doc-context>`. Use doc lookup per `<project-root>/docs/doc-lookup.md` in the project root.
- **Skills**: If planning touches specific domains (testing, frontend, Livewire), invoke the relevant skills. Store as `<skill-context>`.

7. **Brainstorm Technical Approaches**: After inspecting the repo, brainstorm for complex tasks. Complex = touches multiple subsystems, has multiple valid high-level approaches, lacks concrete `<proposed-technical-direction>`, or user flags it. For complex tasks: ask remaining clarifying questions one at a time; propose 2-3 approaches grounded in `<repo-context>` with trade-offs; lead with a recommendation; ask the user to choose one. Record as `<technical-approach-decision>` and get explicit alignment before shaping the spec. For non-complex tasks, skip only when `<proposed-technical-direction>` is concrete and unambiguous and the user has explicitly confirmed it.

8. **Build Action Inventory**: Before writing requirement items, list every concrete action needed to deliver the destination. Use the surface inventory from `<repo-context>`. For each surface (DB, model, rules, services, API, UI, infra, tests, docs, ops), decide `no change` or a concrete action (`create`, `modify`, `delete`, `run`). Record `surface`, `action`, `target`, `depends-on`, `patch-ready` (`true` when the exact code change can be written from `<repo-context>`). Convert each `patch-ready` action into a requirement item in step 9. Every required action must become a requirement item.

9. **Shape the Spec**: Write the spec using the template. Turn `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, `<clarified-requirements>`, `<technical-approach-decision>`, repo findings, `<project-standards>`, `<doc-context>`, and `<skill-context>` into:
- `<spec-title>`: short, useful title.
- `<spec-description>`: the rendered spec template (Destination, Problem Statement, Solution, Notes only — NOT the implementation/validation items). Must include: (a) the destination and why it fixes scope, (b) the chosen technical approach and why, (c) key user preferences and constraints, (d) accepted trade-offs. Make it specific to this spec and user.
- `<requirement-items>`: precise patch descriptions, one per spec step. Map each action from step 8 and each user preference from `<clarified-requirements>` to one or more requirement items.
- `<validation-items>`: validation checklist aligned with project testing conventions. Adhere to `<project-standards>`, `<doc-context>`, `<skill-context>`. Preserve valid technical details; improve incomplete ones when repo inspection provides better direction. Avoid placeholder labels.

**Requirement Item Format**: Each `<requirement-item>` is a self-contained block that an implementation agent can apply as a patch. No item may describe an outcome without naming where the code changes. Required fields: `id` (stable spec-step-id like `S1` or short slug), `file` (repo-root-relative path(s); use a locator rule if multiple candidates), `symbol` (function/class/method/component; fully-qualified when ambiguous), `location` (exact insertion or modification point), `instruction` (one short imperative sentence: `add`/`remove`/`modify`/`replace`), `patch` (minimal code snippet or unified diff; required when target is known; keep to the delta).

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

Rules: One item = one single-purpose patch. Split compound changes into separate items. For broader changes, use sub-items (`S1a`, `S1b`). Verify paths and symbols against `<repo-context>`; no guesses. If you cannot confirm a target, flag it in the item and let the user catch it at step 10 review. No exploratory steps ("investigate X", "consider Y", "ensure X", "update X as needed"). No alternative designs or re-evaluation after the user agrees on direction.

10. **User Review**: Present the spec for approval in a single gate. Reflect the user's inputs back: list 3-5 key decisions or constraints and where they appear (cite requirement/validation items); confirm the spec covers all concrete actions; then ask:
> Spec ready: `<spec-title>`. Implementation: N items. Validation: N items. Want me to sync to the ticket system or make changes?

Wait for the user's response. Revise if asked. Sync only after approval.

11. **Sync Ticket**: Sync the spec to the ticket system via the issue tracker sync tool (see `<project-root>/docs/issue-trackers.md`). Final output: the ticket URL.
- `title`: `<spec-title>`, `description`: `<spec-description>` (Destination/Problem/Solution/Notes only — no implementation/validation items, no patch code).
- `checklists`: two non-empty sections as JSON array matching `{name, items: [{name, completed}]}`:
  - `{"name": "Implementation", "items": [{"name": "<requirement-item-encoded>", "completed": false}, ...]}`
  - `{"name": "Validation", "items": [{"name": "<validation-item>", "completed": false}, ...]}`
  - `<requirement-item-encoded>`: `S1: <title> - <file> @ <symbol>::<location> - <instruction>` (no `patch` code — checklists are short tracking items).
  - Before calling, verify the `checklists` schema against the sync tool's source.
- `patches comment`: post the full structured `<requirement-items>` blocks (with `patch` code) as a comment on the ticket via the issue tracker comment operation (see `<project-root>/docs/issue-trackers.md`). This keeps the implementer's code out of the card body, which overflows ticket systems with card-body limits (e.g. Trello). Skip only when the tracker has no comment support; in that case fall back to attaching the blocks as a file.
- `refUrl`: existing ticket URL → update; no `<ticket-url>` → omit `refUrl`, create new ticket, output the URL.

## Notes

- Ticket load tool downloads attachments to `storage/app/mcp/{source}` — read before planning. Use the ticket list tool to discover unresolved assigned tickets first. During clarification, ask the next question; do not restate previous answers.

