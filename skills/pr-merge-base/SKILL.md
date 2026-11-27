---
name: pr-merge-base
description: "Merge the latest master or main branch into a PR branch and resolve conflicts."
---

## Required `<project-root>/docs` reads

Shell-read before loading PR metadata: `<project-root>/docs/git-hosts.md`. If missing or incomplete, note the gap; use a supported git host or CLI fallback.

## 1. Identify the PR and branches

Resolve the PR from the explicit number/URL, else the unambiguous PR targeted by the session's current work. Missing or ambiguous → ask. Use that PR as the sole target, not the current branch. Load its current state, base repository and branch, head repository and branch, and URL through the git host. If the PR is closed, its head branch is missing, a repository is inaccessible, or metadata is unavailable, stop and report.

Choose the source branch from the base repository: the PR base if it is `master` or `main`; otherwise the repository's default branch if it is `master` or `main`. If neither identifies the source, ask the user which `master` or `main` branch to use. Match Git remotes to the base and head repositories, including forks; if either cannot be identified or fetched, stop rather than guessing `origin`.

## 2. Prepare the PR branch

Inspect the worktree, index, and Git operation state before switching or merging. If there are uncommitted or untracked changes, an existing merge/rebase, or a PR branch checked out in another worktree, stop and preserve them.

Fetch the PR head from its head repository and record its commit → `<pr-head-commit>`. Check out the PR branch at that commit — an existing local `<pr-branch>` counts only when its upstream is the head repository's `refs/heads/<pr-branch>`; a same-named branch without that upstream is a collision → check out a distinct ref at `<pr-head-commit>`, never modify it. An existing local branch behind → sync it yourself: fast-forward to `<pr-head-commit>` — behind alone never blocks. Ahead of it → continue; the push publishes the local commits with the merge — flag them in the report. Diverged → `git rebase <pr-head-commit>` only when the replay range is linear — merge commits can carry edits a clean replay silently drops → a merge in the range → stop; a clean replay — already-upstream commits drop on their own — → continue, flag the replayed commits in the report; conflict → `git rebase --abort`, stop and report. Verify the checked-out branch and that `<pr-head-commit>` is an ancestor of it; record the checked-out tip → `<aligned-tip>`. Never reset, overwrite, or discard local work.

Fetch the chosen source branch from the base repository and record its fetched commit → `<source-commit>`. Already an ancestor of `<pr-head-commit>` → report that the branch is current and stop. Already an ancestor of `<aligned-tip>` → the merge exists locally unpublished — skip to Step 4's approval gate; Step 5 then pushes the tip with nothing to complete.

## 3. Merge and resolve

Prepare `<merge-message>` using the repository's commit convention. Start a non-fast-forward merge of `<source-commit>` into the PR branch with that message and disable automatic committing so even a clean merge stays pending for review and approval. A merge error unrelated to conflicts → stop and report.

If conflicts exist, anchor `<pr-intent>` — why the PR exists: title, body, and, when the PR references issues or cards, a ticket brief (see Ticket Brief) scoped to intended behavior and constraints. For each scope-relevant target PR decision reference returned by the brief, reuse loaded feedback or retrieve only that decision and the context needed to establish its status via the git host (`git-hosts.md`); unavailable or ambiguous → note the gap. No usable brief → fall back to title/body plus any verified decision; note the gap.

Investigate each conflict using the merge base, both sides, and the surrounding behavior. Preserve the intent of both changes — judge the PR side against `<pr-intent>`; do not blindly select one side. Resolve and stage each affected path, including deletions and renames. The base removing or replacing behavior `<pr-intent>` assumes is premise drift → a product decision per below, never a silent resolution.

If a conflict requires a product decision or cannot be resolved safely, leave the merge pending and ask the user. Only with explicit approval to abort, abort the pending merge and stop without pushing.

## 4. Verify and approve the batch

Confirm the index has no unmerged entries, inspect any staged merge for conflict markers and unintended changes, and run relevant checks for the resolved files. Report failed or unavailable checks. If validation fails, stop before approval and leave any merge pending.

Show the PR URL, source and PR branches, verified push destination `<head-repository>:<pr-branch>` (fork heads included), resolved paths, verification results, and any existing local PR-branch commits the push would publish.

Ask once (header `Approval`): "Approve this batch for <pr-url>: finish the pending merge (if any) and push to <head-repository>:<pr-branch>?" Offer `Go Ahead` and `Needs Review`. Only `Go Ahead` approves. An unanswered or declined gate stops publication.

`Go Ahead` authorizes Step 5 for the shown batch: finish the merge if needed, then push without asking again for either action. Keep the approval when switching to a documented fallback; honor tool-enforced permissions and higher-priority safety gates. If the target or batch changes materially, re-verify and return to this gate before publishing. `Needs Review` → refine within scope, re-verify, and present the batch again.

## 5. Finish and push

On `approved`, re-fetch the PR head first; if it changed since preparation, stop and report. If Step 3 started a merge, confirm `HEAD` is `<aligned-tip>` and `MERGE_HEAD` is `<source-commit>`. Missing or mismatched state → stop and report. Continue the pending merge with the editor disabled for that command so Git accepts the prepared message. Do not create a separate commit. Continuation fails → report and stop without pushing. Before pushing, require exactly two parents in order: `<aligned-tip>`, `<source-commit>`. If Step 2 found the source already in the local tip, push that tip without creating a commit. Push normally to the verified head repository and PR branch, never force-push. Report the merge commit or pre-existing tip and the pushed ref.

## Ticket Brief

Reuse current, scoped evidence; otherwise use `issue-trackers.md` to dedupe by tracker/ID, read only scoped fields, and read the latest comment page for behavior or constraints. Follow older material or other links only to settle a gap; for the target PR return decision references without fetching feedback. Return cited answers, decision status (confirmed/proposed/superseded), gaps and blockers; mark partial answers and leave tracker state unchanged. Read inline, or delegate one bounded read-only brief when it saves distracting context or the user requests it; pass the refs, scope, `<project-root>` and target PR. Worker unavailable → inline; completed lookup blocker → do not retry. Missing spec → report the gap.
