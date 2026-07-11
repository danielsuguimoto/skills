---
name: to-specs
description: Turn a request or ticket into a spec, then sync to the ticket system. Read-only (no code changes or implementation).
---

Turn a request or ticket into a spec, then sync to the ticket system. Read-only (no code changes or implementation). Clarify requirements with the user before shaping the spec. No assumptions, guesses, or inferred intent. Treat ticket systems generically. Use `<additional-context>` for constraints and focus areas. Update existing tickets; create replacements only when asked. For big problems, name the destination first and keep all output on one ticket. Split into multiple tickets only when the user explicitly asks.

<spec-template>

## Destination — the spec, decision, or change this effort delivers. One or two lines.
## Problem Statement — the user-facing problem or opportunity that drives the spec.
## Solution — the intended outcome from the user's perspective. Record each locked decision from grilling or repo inspection on one line with its source.
## Implementation Items — numbered self-contained patches. Each item has `file`, `symbol`, `location`, `instruction`, and `patch`. Include every concrete action: code, schema, config, tests, operations.
## Validation Items — checklist of concrete verifications. Each item maps to an implementation item or user-facing behavior.
## Notes — other context, dependencies, or decisions worth keeping.

</spec-template>

1. **Interpret Arguments**: Map a ticket reference/URL to `<ticket-url>`. Map any other input to `<request>`. Map extra focus areas or constraints to `<additional-context>`. Derive `<request>` from the conversation when the user provides no arguments.

2. **Load Planning Context**: If `<ticket-url>` exists, load ticket context via the issue tracker tool (see `/docs/issue-trackers.md` in the project root) and store as `<planning-context>`. Read each attachment with `relativePath` via `read`. Describe images, extract key requirements from documents into `<attachment-insights>`, note inaccessible attachments. If `<ticket-url>` is absent, treat the request or conversation as `<planning-context>`. If empty or missing, stop.

3. **Interpret Planning Context**: Derive from the full ticket (raw text + comments): `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, and `<open-questions>` (only unresolved issues). Keep earlier comments that define constraints, business rules, implementation decisions, migration rules, naming, sequencing, or scoping. If the request describes multiple independent subsystems, flag it immediately. Help the user identify the independent pieces, their relationships, and the build order. Plan the first subsystem through the normal flow. Split into separate tickets only when the user explicitly asks.

4. **Gather Project Standards**: Read the relevant `AGENTS.md` files and `/docs/` specifications in the project root (project root and module-specific). Check the active memory provider for code standards, architecture, and tech stack. Store as `<project-standards>`.

5. **Clarify Requirements with User (MANDATORY)**: Run a clarification interview before shaping the spec by default. Skipping is the exception; justify it in the spec output. No assumptions, guesses, or inferred intent instead of user clarification.

Skip gate (all must be true): single atomic edit with no design decisions; `<proposed-technical-direction>` concrete and unambiguous; no earlier comments conflict; no open questions about scope, naming, placement, ordering, or behavior; repo inspected and direction matches existing patterns; user explicitly confirmed in the current conversation.

If the request is unclear or any gate is false or uncertain, invoke a clarification interview. Pass planning context, `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, `<open-questions>`, and relevant module path(s). Store resolved decisions as `<clarified-requirements>`. Continue only after shared understanding. Reflect `<clarified-requirements>` in `<spec-description>`, `<requirement-items>`, or `<validation-items>`. Do not proceed to step 6 until you capture all user inputs.

Skip reason (when used): `Skip reason: <all 6 conditions met because ...>`. Skipping without justification is a process violation.

6. **Gather Context** (repo, docs, skills): Delegate each layer to the matching subagent. Fall back to inline when unavailable or `BLOCKED`.
- **Repo**: Delegate to a subagent with `<planning-objective>`, `<proposed-technical-direction>`, `<related-entities>`, module path. Require entry points, existing patterns, callers, schema facts, `file:line` citations, validation of `<proposed-technical-direction>`, and a **surface inventory** of each layer the change touches (DB, model, policy, service, controller, API, UI, tests, config, docs, ops). For each layer state: patch, new file, or no change, with concrete file(s) and symbol(s). Store as `<repo-context>`. Inline fallback: `grep`/`find_file_by_name` for names, code navigation tool (see `/docs/code-navigation.md` in the project root) for symbols/references, database tools (see `/docs/database-tools.md` in the project root) for schema. Read one or two key files. Confirm current behavior and patterns. Note gaps; avoid false certainty.
- **Docs**: Delegate to a subagent with libraries, frameworks, Laravel features, `<proposed-technical-direction>`, API/best-practice questions. Require API signatures, version-specific behavior, gotchas. Store as `<doc-context>`. Inline fallback: doc lookup per `/docs/doc-lookup.md` in the project root.
- **Skills**: If planning touches specific domains (testing, frontend, Livewire), invoke the relevant skills. Store as `<skill-context>`.

7. **Brainstorm Technical Approaches**: After inspecting the repo, brainstorm for complex tasks. Complex = touches multiple subsystems, has multiple valid high-level approaches, lacks concrete `<proposed-technical-direction>`, or user flags it. For complex tasks: ask remaining clarifying questions one at a time; propose 2-3 approaches grounded in `<repo-context>` with trade-offs; lead with a recommendation; ask the user to choose one. Record as `<technical-approach-decision>` and get explicit alignment before shaping the spec. For non-complex tasks, skip only when `<proposed-technical-direction>` is concrete and unambiguous and the user has explicitly confirmed it.

8. **Build Action Inventory**: Before writing requirement items, list every concrete action needed to deliver the destination. Use the surface inventory from `<repo-context>`. For each surface (DB, model, rules, services, API, UI, infra, tests, docs, ops), decide `no change` or a concrete action (`create`, `modify`, `delete`, `run`). Record `surface`, `action`, `target`, `depends-on`, `patch-ready` (`true` when the exact code change can be written from `<repo-context>`). Convert each `patch-ready` action into a requirement item in step 9. If a required action is not `patch-ready`, do not finalize the spec. Return to step 7, 6, or 5 until the target is resolvable. Every required action must become a requirement item.

9. **Shape the Spec**: Write the spec using the template. Turn `<planning-objective>`, `<operative-constraints>`, `<proposed-technical-direction>`, `<clarified-requirements>`, `<technical-approach-decision>`, repo findings, `<project-standards>`, `<doc-context>`, and `<skill-context>` into:
- `<spec-title>`: short, useful title.
- `<spec-description>`: the rendered spec template. Must include: (a) the destination and why it fixes scope, (b) the chosen technical approach and why, (c) key user preferences and constraints, (d) accepted trade-offs. Make it specific to this spec and user.
- `<requirement-items>`: precise patch descriptions, one per spec step. Map each action from step 8 and each user preference from `<clarified-requirements>` to one or more requirement items. If you cannot express any concrete action as a requirement item, the spec is incomplete; return to step 8 or 7.
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

Rules: One item = one single-purpose patch. Split compound changes into separate items. For broader changes, use sub-items (`S1a`, `S1b`). Verify paths and symbols against `<repo-context>`; no guesses. If you cannot confirm a target, mark not `patch-ready` and do not finalize. No exploratory steps ("investigate X", "consider Y", "ensure X", "update X as needed"). No alternative designs or re-evaluation after the user agrees on direction.

10. **User Review**: Before self-review, reflect the user's inputs back: list 3-5 key decisions or constraints and where they appear (cite requirement/validation items); confirm the spec covers all concrete actions; ask if preferences were captured correctly. Wait for confirmation. Revise and repeat if needed. Then run self-review and present the spec:
> Spec ready: `<spec-title>`. Implementation: N items. Validation: N items. Want me to sync to the ticket system or make changes?

Wait for the user's response. Revise and re-run self-review if needed. Sync only after approval.

**Self-Review Checklist** (fix failures inline before syncing):
- [ ] No placeholders, TBDs, TODOs, or vague requirements.
- [ ] Internal consistency: checklists match description; each implementation item has validation coverage.
- [ ] Scope: one ticket. Narrow the destination if not.
- [ ] Code-target precision: each `<requirement-item>` has concrete `file`, `symbol`, `location` against `<repo-context>`.
- [ ] Vertical slices: trace each item through schema → API → UI → tests; avoid horizontal layers.
- [ ] No-replan: no alternative designs, re-evaluation, or exploratory steps.
- [ ] Complete action inventory: every concrete action is a requirement item.
- [ ] Patch coverage: each requirement item with a known target includes a `patch`.
- [ ] Cross-cutting coverage: scaffolding (migrations, seeders, config, feature flags, permissions, translations, observers, events, jobs, cache, search, docs, tests) where impacted.
- [ ] No hidden discovery: no item assumes the implementer will 'find out', 'investigate', or 'determine' something later.

11. **Sync Ticket**: CRITICAL: Sync the spec to the ticket system. Final output: the ticket URL. See `/docs/issue-trackers.md` in the project root for the sync tool, provider-specific parameters, board/list/assignee config, and `checklists` schema.

Sync via the issue tracker sync tool (see `/docs/issue-trackers.md` in the project root):
- `title`: `<spec-title>`
- `description`: `<spec-description>`
- `checklists`: exactly two non-empty sections. JSON array matching `{name, items: [{name, completed}]}`:
  - `{"name": "Implementation", "items": [{"name": "<requirement-item-encoded>", "completed": false}, ...]}`
  - `{"name": "Validation", "items": [{"name": "<validation-item>", "completed": false}, ...]}`
  - `<requirement-item-encoded>`: `S1: <title> - <file> @ <symbol>::<location> - <instruction>`. Append the full structured block (with `patch`) to `<spec-description>` so the implementation agent has the complete patch.
  - Before calling, verify the `checklists` schema against the sync tool's source.
- `refUrl`: existing ticket URL → update; no `<ticket-url>` → omit `refUrl`, create new ticket, output the URL.

## Notes

- The ticket load tool downloads attachments to `storage/app/mcp/{source}`. Read them before planning.
- Use the ticket list tool to discover unresolved assigned tickets first (see `AGENTS.md` → Ticket Tools).
- During the clarification interview, ask the next question; do not restate previous answers.

