---
name: post-merge
description: "Verify deployed PR artifacts, draft customer ticket resolutions, and clear the isolated workspace after merge and deployment."
---

## Required `<project-root>/docs` reads

Shell-read before post-merge cleanup (reuse session reads): `<project-root>/docs/git-hosts.md`, `<project-root>/docs/issue-trackers.md`, `<project-root>/docs/workspace-isolation.md`; only when the PR ships data artifacts — `<project-root>/docs/database-tools.md`. If a spec is missing, report the gap and use a documented fallback.

Explicitly invoking `post-merge` authorizes verified spec-ticket closures, private draft notes, and recoverable cleanup of the workspace for that PR. Reuse that authorization across steps, tickets, and subskills. Follow tool-enforced permissions and the cleanup safeguards below.

### 1. Interpret Arguments

Explicit PR number/URL → `<pr-ref>`; extra guidance → `<additional-context>`. Without an explicit reference, use the unambiguous PR targeted by the session's current work; ambiguous → ask. No session target → use the most recently merged PR on the current branch (git host list PRs, merged state); unavailable or ambiguous → ask.

### 2. Load PR Context

View PR via git host (`git-hosts.md`). Extract `<pr-branch>`, `<pr-number>`, `<pr-url>`, `<pr-summary>` (what shipped and why — title, body, commits), and `<ticket-refs>` (every issue/card reference in body, commits, branch name — git host or external tracker). Inspect the PR file list and diff, deployment configuration, and deploy notes for `<data-artifacts>`: every shipped migration, seeder, operation, or script that changes production schema or data, including changes to existing artifacts. Determine each artifact's deployment targets, intended effects, and whether it is expected to run in this deploy from its code and configuration; record unknowns rather than treating them as absent.

**Gate:** PR must be merged. Open or closed-unmerged → STOP, report its actual state — this flow only applies to shipped work.

### 3. Load Linked Tickets

Per `<ticket-refs>` entry, load fresh via the issue tracker tool (`issue-trackers.md`): body, checklists, full comment thread, attachments — an earlier in-session copy does not qualify; resolution drafts must reflect the latest customer comments. This fresh snapshot is what `draft-reply` reuses. Loads are independent — three or more tickets, or fewer on the user's explicit request, may be loaded in parallel when a suitable subagent profile exists; doubtful independence stays serial. Each worker returns the snapshot plus nested refs found, and the parent dedupes new refs against `<ticket-refs>` before loading them. **Nested links:** traverse per the spec — the customer thread may live on an inner card; add each ref found to `<ticket-refs>`.

### 4. Validate Deployed Artifacts

Use linked ticket context where relevant to identify deployment targets, intended effects, schedules, and verification evidence. Skip only when the PR ships no data artifacts. If artifact discovery is incomplete, record `unverifiable` with the gap instead of `none shipped`. Per artifact and production target, record `not scheduled` for confirmed manual or deferred work not expected in this deploy. If ticket schedules conflict with code or deployment configuration, resolve the conflict before recording `not scheduled`; until resolved, treat the schedule as unknown and query production. For every other pair, query the live production database per `database-tools.md`; keep production read-only. Artifact-target pairs are independent read-only checks — three or more pairs, or fewer on the user's explicit request, may run in parallel as read-only workers when a suitable subagent profile exists; doubtful independence stays serial. Check execution evidence and the intended effect:

- Migration → migration history records its run; inspect the resulting schema and data.
- Seeder → inspect execution records or durable run markers and the expected records or updated values.
- Operation/script → inspect execution records or durable run markers and the stated effect (rows written, values changed, flags set).

Status each pair as `verified` only when database evidence establishes execution for this deploy and the intended effect, and the run's final outcome is successful; `failed` when a run reaches a final outcome but its effect is incomplete, or when an expected run was definitively skipped. Otherwise `unverifiable`, including an unknown schedule without evidence of execution. Matching state alone does not prove execution: pre-existing data can mask a skipped run. After attempting database checks, if execution or completion remains unclear, shell-read `observability-tools.md` and inspect available deploy or job records; a confirmed terminal failure for the same artifact, deploy, and target → `failed`. After a run completes, if the data was read before completion or its timing is unknown, re-query a fresh production database snapshot before assessing the effect. Consider retries: a failed attempt superseded by a successful retry is not a terminal failure; a running job or unknown final outcome remains `unverifiable`. Logs do not replace production database checks. If missing access, unknown targets, or insufficient execution evidence prevent a determination, record `unverifiable` and the gap. Record findings and evidence as `<artifact-results>`.

### 5. Classify Tickets

- **Customer-success ticket** → `<customer-tickets>` — filed in the customer-facing tracker where customers report issues (per the spec), or carries a customer requester/thread. Resolution replies belong here.
- **Internal ticket** → `<internal-tickets>` — planning cards, spec tickets, dev tasks. No customer reply.
- **Spec ticket** → `<spec-tickets>` — an internal ticket that is the implementation spec for the shipped work: linked as the spec destination from another `<ticket-refs>` entry, or carrying the spec body sections (Destination/Problem Statement/Solution/Notes) or Implementation/Validation tracking for the same change. Spec tickets also belong to `<internal-tickets>`.
- **Premise drift** — a customer-success ticket whose reported problem `<pr-summary>` doesn't cover (the PR shipped a different fix, or the issue was superseded) → `<drifted-tickets>` — no resolution draft; reported per ticket in Step 9.

Ambiguous → ask the user rather than drafting to the wrong audience or closing the wrong ticket.

### 6. Close Spec Tickets

Skip an already closed/completed/done `<spec-tickets>` entry as `already done`. For every open entry, first match the production data or schema effects its spec requires against `<artifact-results>`; tickets without those effects have no artifact dependency. Leave the ticket unchanged and record `blocked` when such a required effect is `failed`, `not scheduled`, `unverifiable`, or absent from `<artifact-results>`; never block it on an unrelated artifact. Otherwise close it or mark it done through the issue tracker update operation (`issue-trackers.md`). Never close customer-success tickets or non-spec internal tickets here. Record `done` only when the update response or a fresh reload confirms that terminal state; `unsupported` when the tracker lacks closure/completion; `failed` when the update errors or the outcome stays unconfirmed. Store per-ticket URL, status, and error, gap, or blocking artifact and target as `<spec-ticket-results>`.

**Gate:** any `failed` in `<artifact-results>` → STOP; report per Step 9 with customer replies and workspace cleanup `not run` — spec tickets are already accounted, but those steps wait until the artifact re-runs. `unverifiable` → proceed; flag per artifact in Step 9.

### 7. Draft Resolution Replies

Per `<customer-tickets>` entry, invoke `draft-reply` with the ticket URL and `<additional-context>` carrying:

- The fix shipped: merged in `<pr-url>`, now deployed — the reported issue should be resolved.
- What the issue was: plain-language explanation from `<pr-summary>` + the thread — include **only when nobody explained it to the customer before** (check comments first); describe symptom → cause → fix at their level, no root-cause jargon.
- Caveats: each `unverifiable` or `not scheduled` entry in `<artifact-results>` → `[NEEDS-REVIEW: <what couldn't be confirmed or isn't scheduled>]` — a reviewer flag in the draft, never customer-facing copy.

One draft per ticket — `draft-reply` posts a private note or returns the unposted draft and reason; nothing reaches the customer. Relay each unposted draft and reason to the user. Invocations are independent per ticket — three or more, or fewer on the user's explicit request, may run in parallel when a suitable subagent profile exists; each subagent runs in the foreground (a background worker can silently lose the note when posting requires approval), is delegated `draft-reply` for exactly one ticket, is front-loaded with the parent's fresh snapshot for that ticket so it does not reload, and posts only there.

### 8. Clear the Workspace

Identify the isolated workspace created for this PR via `workspace-isolation.md`. No match → report `none found`; ambiguous match → leave workspaces intact and report the gap. Verify that removal preserves local-only commits, modified files, untracked or ignored data, and any other content that cannot be recovered. If recovery is verified, run the documented removal without another approval and verify the result. If removal would irreversibly discard data, describe the exact operation and workspace path and obtain specific confirmation before running it. If you cannot establish the effects, leave the workspace intact and report why.

### 9. Report

Chat only: `<pr-url>` merge state · `<artifact-results>` per artifact and target — status + evidence, plus any inventory-wide discovery gap, or `none shipped` · classification counts · per customer ticket, the draft note URL, unposted draft and reason, or `not run` · `<spec-ticket-results>` per entry: ticket URL + status + error, gap, or blocking artifact and target, or `none linked` · `<drifted-tickets>` per entry: ticket URL, reported problem, why `<pr-summary>` doesn't cover it · workspace removed, `none found`, `blocked` with reason, or `not run`.

## Notes

- Deployed state comes from the trigger — the user says it's deployed; Step 2 verifies merged state, Step 4 what the deploy executed.
