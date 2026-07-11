---
name: feedback
description: "Apply one round of local code-review feedback after implement. Fixes/improves code only — no PR, no commit, no push, no next steps, no loop."
argument-hint: "[feedback]"
---

# Feedback

Apply the user's local code-review feedback to the implementation. Local only — no PR, no threads, no commit, no push, no approval-driven next steps.

## Scope Boundary

Act on feedback only: fix bugs, apply improvements, refactor per notes. Do NOT:
- Create or update PRs
- Commit or push (`git commit`, `git push`, `git-ops` commit tools)
- Reply to review threads (none exist)
- Suggest follow-up work, next steps, or shipping on completion
- Widen scope unless feedback explicitly requests it

The user ships separately.

## Quick Reference

- Input: `$ARGUMENTS` = user feedback (free-form prose, possibly multi-point).
- Branch: stay on the current branch (typically `master`). Do NOT switch.
- Tools: `git-ops` MCP for status/diff only (fallback `git` CLI). Never use commit/push tools.
- Do NOT delegate to PR-shaped subagents — they expect PR metadata and misinterpret local feedback. Parse inline; spawn generic background subagents for parallel groups only if the runtime supports them.

### 1. Capture Feedback

Read `$ARGUMENTS` → `<feedback-raw>`. If empty or only a question, STOP and ask for concrete feedback.

### 2. Load Current Changes

Get workspace state:
- `git-ops` `change_scan` (fallback `git status --short` + `git diff --stat`) → `<changes>`.
- Clean tree (no implementation changes) → STOP and tell the user there is nothing to apply feedback against.

Store `<active-branch>` = `git branch --show-current`. Do NOT checkout.

### 3. Parse Feedback Into Actionable Items

Parse `<feedback-raw>` inline — do NOT delegate to PR-shaped subagents. If feedback is large (≥8 points) AND the runtime supports subagents, you may delegate to a generic background subagent framed as: "local post-implementation feedback, no PR, parse into the structured format below" — otherwise inline.

Inline parse:

1. Split `<feedback-raw>` into discrete points. Each → one `<feedback-item>`:
   - `id`: `F1`, `F2`, …
   - `file`: target file(s) if locatable from feedback + `<changes>`; else `TBD`
   - `lines`: range if stated; else `TBD`
   - `category`: `critical` (bug/security/broken contract) | `important` (behavior/correctness) | `style` (naming/format/clarity)
   - `suggestion`: one-line imperative
   - `current-heading`: `yes` if feedback contradicts the implementation's intended direction and the implementation should be defended (record reason; do NOT apply)
2. Build `<not-fixing>` list for `noise`, `question`, or `current-heading: yes` items with one-line reasons.

### 4. Plan Implementation

Fast path: every actionable `<feedback-item>` is a specific code suggestion with no conflicts → `<implementation-plan>` = `trivial`, skip to Step 5 serial. Otherwise:

1. Order: `critical` → `important` → `style`, then file, then line.
2. Validate each: mark `invalid` if it would break behavior or contradicts ticket intent → `<not-fixing>` with reason.
3. Map cross-item dependencies (shared files/types/callers) → `<dependencies>`.
4. Identify file-disjoint groups for parallel execution.
5. Pick path: Parallel (≥2 file-disjoint groups, no cross-group deps) or Serial.

### 5. Implement Fixes

Parallel: spawn generic background subagents per group (if the runtime supports them). Pass the group's `<feedback-item>`s, file paths, and the constraint "apply only these fixes, no PR, no commits, stay on current branch". Await and merge results. Parent handles cross-file/ambiguous items serially. Serial: fix critical first, follow existing patterns, make minimal edits, no comments unless asked, stay on `<active-branch>`.

After either: store `<changes-count>`.

### 6. Validate Changes

Run the project's lint and test commands. If the runtime supports subagents and the suite is slow, delegate both to a generic background subagent in one task; otherwise inline. The subagent returns combined pass/fail + failure traces. Store `<validation-results>` and `<validation-passing>` (`yes`/`no`).

Confirm fixes address the feedback points. If a fix misses intent, re-implement that item.

### 7. Present Results

No commit/push approval gate — nothing ships. Present a concise summary and stop.

- One line per applied fix.
- Validation status (`<validation-passing>` + `<validation-results>` summary).
- `<not-fixing>` list with one-line reasons.
- `<changes-count>` if tracked.

No follow-up questions, no refinement loop, no next steps.

Output the summary of changes applied and validation results.

STOP. No next steps, no shipping suggestions, no PR creation. The user invokes other skills to ship.

## Red Flags

**Never:**
- Commit, push, or call any commit/push MCP/CLI tool
- Create a PR, comment on a PR, or reply to review threads
- Switch branches or checkout anything
- Widen scope beyond the feedback
- Suggest follow-up work or shipping on completion
- Skip validation after implementing
- Apply feedback marked `current-heading: yes` without explicit user confirmation

**Always:**
- Stay on `<active-branch>`
- Drive fixes off parsed `<feedback-item>`s
- Run lint + tests after changes
- Present results and stop — no unsolicited next steps

