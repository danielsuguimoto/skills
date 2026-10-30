---
name: post-merge
description: "Use after a pull request is merged and deployed — find customer-success tickets linked to the PR and draft resolution replies for them, then clear the isolated workspace used for the work. No code changes."
---

## Required `<project-root>/docs` reads

Read via shell before post-merge cleanup — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/git-hosts.md`
- `<project-root>/docs/issue-trackers.md`
- `<project-root>/docs/workspace-isolation.md`

Drafts go to a human for review.

### 1. Interpret Arguments

PR number/URL → `<pr-ref>`; none → derive from conversation or the most recently merged PR on the current branch (git host list PRs, merged state); extra guidance → `<additional-context>`.

### 2. Load PR Context

View PR via git host (`<project-root>/docs/git-hosts.md`). Extract `<pr-branch>`, `<pr-number>`, `<pr-url>`, `<pr-summary>` (what shipped and why — title, body, commits), `<ticket-refs>` (every issue/card reference in body, commits, branch name — git host or external tracker).

**Gate:** PR must be merged. Open or closed-unmerged → STOP, report its actual state — this flow only applies to shipped work.

### 3. Load Linked Tickets

Per `<ticket-refs>` entry, load via the issue tracker tool (`<project-root>/docs/issue-trackers.md`): body, every comment, attachments. **Nested links:** traverse per the spec — the customer thread may live on an inner card; add each ref found to `<ticket-refs>`.

### 4. Classify Tickets

- **Customer-success ticket** → `<customer-tickets>` — filed in the customer-facing tracker where customers report issues (per the spec), or carries a customer requester/thread. Resolution replies belong here.
- **Internal ticket** → `<internal-tickets>` — planning cards, spec tickets, dev tasks. No customer reply.

Ambiguous → ask the user (interactive question tool when one exists) rather than drafting to the wrong audience.

### 5. Draft Resolution Replies

Per `<customer-tickets>` entry, invoke `draft-reply` with the ticket URL and `<additional-context>` carrying:

- The fix shipped: merged in `<pr-url>`, now deployed — the reported issue should be resolved.
- What the issue was: plain-language explanation from `<pr-summary>` + the thread — include **only when nobody explained it to the customer before** (check comments first); describe symptom → cause → fix at their level, no root-cause jargon.

One draft per ticket — `draft-reply` posts a private note; nothing reaches the customer.

### 6. Clear the Workspace

List isolated workspaces per `<project-root>/docs/workspace-isolation.md`; find the one created for this PR (match `<pr-branch>` or the ticket slug); remove via the spec's remove operation. Confirm removal — never delete a workspace not tied to this PR. No match → report `none found`, move on.

### 7. Report

Chat only: `<pr-url>` merge state · classification counts · draft note URL per customer ticket · workspace removed or `none found`.

## Notes

- Deployed state comes from the trigger — the user says it's deployed; only the merged check (Step 2) is verified.
