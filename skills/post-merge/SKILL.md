---
name: post-merge
description: "Use after a pull request is merged and deployed — find customer-success tickets linked to the PR and draft resolution replies for them, then clear the isolated workspace used for the work. No code changes."
---

## Required `<project-root>/docs` reads

Read these project-root spec files before post-merge cleanup (use shell `cat`/`ls` — they may be in `.gitignore`, invisible to built-in search). Missing file → fall back to native tools, note the gap; never invent contents.

- `<project-root>/docs/git-hosts.md`
- `<project-root>/docs/issue-trackers.md`
- `<project-root>/docs/workspace-isolation.md`

Close the loop on shipped work: tell customer-facing tickets their issue is resolved, then tear down the workspace the work happened in. No code changes, no commits. Drafts go to a human for review — the agent never replies to a customer directly.

### 1. Interpret Arguments

- PR number/URL → `<pr-ref>`; no arguments → derive from conversation or the most recently merged PR on the current branch (git host list PRs with merged state, see `<project-root>/docs/git-hosts.md` in the project root); extra guidance → `<additional-context>`.

### 2. Load PR Context

View the PR via git host view PR (see `<project-root>/docs/git-hosts.md`). Extract `<pr-branch>`, `<pr-number>`, `<pr-url>`, `<pr-summary>` (what shipped and why — from title, body, commits), and `<ticket-refs>` (every issue/card reference in the body, commits, or branch name — on the git host or any external tracker).

**Gate:** the PR must be merged. Open or closed-unmerged → STOP and report its actual state; the deploy-and-notify flow only applies to shipped work.

### 3. Load Linked Tickets

For each `<ticket-refs>` entry, load the ticket via the issue tracker tool (see `<project-root>/docs/issue-trackers.md`): body, every comment, attachments.

**Follow nested links.** A PR commonly links to a git host issue that itself links to a card on another platform — the customer thread may live on the inner card. After loading each ticket, scan its body, comments, and attachments for references to other issues/cards on any platform. Load each new reference, repeat the scan on it, and add it to `<ticket-refs>` until no new refs surface. Dedupe visited refs so cross-linked tickets don't loop.

### 4. Classify Tickets

Split loaded tickets into `<customer-tickets>` and `<internal-tickets>`:

- **Customer-success ticket** — filed in the customer-facing tracker where customers report issues (per `<project-root>/docs/issue-trackers.md`), or carries a customer requester/thread. This is where a resolution reply belongs.
- **Internal ticket** — planning cards, spec tickets, dev tasks. No customer reply.

Ambiguous classification → ask the user rather than drafting to the wrong audience.

### 5. Draft Resolution Replies

For each `<customer-tickets>` entry, invoke `draft-reply` with the ticket URL and `<additional-context>` carrying:

- The fix shipped: merged in `<pr-url>`, now deployed — the reported issue should be resolved.
- What the issue was: a plain-language explanation derived from `<pr-summary>` and the ticket thread — include it **only when nobody has explained the issue to the customer before** (check comments first; skip if the thread already carries one). No root-cause jargon the customer can't act on; describe symptom → cause → fix at their level.

One draft per ticket. `draft-reply` posts a private note — nothing reaches the customer; a human sends it.

### 6. Clear the Workspace

List isolated workspaces per `<project-root>/docs/workspace-isolation.md` and find the one created for this PR (match `<pr-branch>` or the ticket slug). Remove it via the spec's remove operation. Confirm removal; never delete a workspace that isn't tied to this PR. No matching workspace → report `none found`, move on.

### 7. Report

Chat only: `<pr-url>` merge state · tickets classified (`<customer-tickets>` count, `<internal-tickets>` count) · draft note posted per customer ticket (ticket URL each) · workspace removed or `none found`.

## Notes

- Deployed state comes from the trigger — the user says it's deployed. Only the merged check (Step 2) is verified.
