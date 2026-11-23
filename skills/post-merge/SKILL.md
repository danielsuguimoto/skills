---
name: post-merge
description: "Use after a pull request is merged and deployed — verify shipped artifacts took effect, find customer-success tickets linked to the PR and draft resolution replies for them, then clear the isolated workspace used for the work. No code changes."
---

## Required `<project-root>/docs` reads

Shell-read before post-merge cleanup; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`, `<project-root>/docs/issue-trackers.md`, `<project-root>/docs/workspace-isolation.md`; only when the PR ships data artifacts — `<project-root>/docs/database-tools.md`, `<project-root>/docs/observability-tools.md`

Drafts go to a human for review.

### 1. Interpret Arguments

PR number/URL → `<pr-ref>`; none → derive from conversation or the most recently merged PR on the current branch (git host list PRs, merged state); extra guidance → `<additional-context>`.

### 2. Load PR Context

View PR via git host (`git-hosts.md`). Extract `<pr-branch>`, `<pr-number>`, `<pr-url>`, `<pr-summary>` (what shipped and why — title, body, commits), `<ticket-refs>` (every issue/card reference in body, commits, branch name — git host or external tracker), `<data-artifacts>` (shipped migrations, seeders, operations, and scripts the deployment executes — from the PR file list, body, and deploy notes).

**Gate:** PR must be merged. Open or closed-unmerged → STOP, report its actual state — this flow only applies to shipped work.

### 3. Validate Deployed Artifacts

Skip when `<data-artifacts>` is empty. Per artifact, derive its intended effect from the shipped code, then verify in production that it ran in this deploy and took effect — keep production read-only:

- Migration → its run recorded for this deploy and its schema/data change present.
- Seeder → run evidence tied to this deploy and expected records or updated values present.
- Operation/script → run evidence tied to this deploy and its stated effect observable (rows written, values changed, flags set).

Evidence: deploy and job logs per `observability-tools.md` tie the run to this deploy; live-DB queries per `database-tools.md` prove the outcome. Matching state with no run evidence → `unverifiable`, never `verified` — pre-existing data masks a skipped run. Status each `verified` / `failed` / `unverifiable` → `<artifact-results>`.

**Gate:** any `failed` → STOP, report — the shipped work is unproven until the artifact re-runs. `unverifiable` → proceed; flag per artifact in Step 8.

### 4. Load Linked Tickets

Per `<ticket-refs>` entry, load fresh via the issue tracker tool (`issue-trackers.md`): body, every comment, attachments — an earlier in-session copy does not qualify; resolution drafts must reflect the latest customer comments. This fresh snapshot is what `draft-reply` reuses. **Nested links:** traverse per the spec — the customer thread may live on an inner card; add each ref found to `<ticket-refs>`.

### 5. Classify Tickets

- **Customer-success ticket** → `<customer-tickets>` — filed in the customer-facing tracker where customers report issues (per the spec), or carries a customer requester/thread. Resolution replies belong here.
- **Internal ticket** → `<internal-tickets>` — planning cards, spec tickets, dev tasks. No customer reply.
- **Premise drift** — a customer-success ticket whose reported problem `<pr-summary>` doesn't cover (the PR shipped a different fix, or the issue was superseded) → `<drifted-tickets>` — no resolution draft; reported per ticket in Step 8.

Ambiguous → ask the user rather than drafting to the wrong audience.

### 6. Draft Resolution Replies

Per `<customer-tickets>` entry, invoke `draft-reply` with the ticket URL and `<additional-context>` carrying:

- The fix shipped: merged in `<pr-url>`, now deployed — the reported issue should be resolved.
- What the issue was: plain-language explanation from `<pr-summary>` + the thread — include **only when nobody explained it to the customer before** (check comments first); describe symptom → cause → fix at their level, no root-cause jargon.
- Caveats: each `unverifiable` entry in `<artifact-results>` — name what couldn't be confirmed, so the reviewer sees the gap beside the resolution claim.

One draft per ticket — `draft-reply` posts a private note; nothing reaches the customer.

### 7. Clear the Workspace

List isolated workspaces per `workspace-isolation.md`; find the one created for this PR (match `<pr-branch>` or the ticket slug); remove via the spec's remove operation. Confirm removal — never delete a workspace not tied to this PR. No match → report `none found`, move on.

### 8. Report

Chat only: `<pr-url>` merge state · `<artifact-results>` per artifact — status + evidence, or `none shipped` · classification counts · draft note URL per customer ticket · `<drifted-tickets>` per entry: ticket URL, reported problem, why `<pr-summary>` doesn't cover it · workspace removed or `none found`.

## Notes

- Deployed state comes from the trigger — the user says it's deployed; Step 2 verifies merged state, Step 3 what the deploy executed.
