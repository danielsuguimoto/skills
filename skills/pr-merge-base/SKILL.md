---
name: pr-merge-base
description: "Use when merging the latest master or main branch into a pull request branch, including conflict resolution."
---

## Required `<project-root>/docs` reads

Shell-read before loading PR metadata: `<project-root>/docs/git-hosts.md`. If missing or incomplete, note the gap; use a supported git host or CLI fallback.

## 1. Identify the PR and branches

Resolve the PR from the explicit number/URL, else the unambiguous PR targeted by the session's current work. Missing or ambiguous → ask. Use that PR as the sole target, not the current branch. Load its current state, base repository and branch, head repository and branch, and URL through the git host. If the PR is closed, its head branch is missing, a repository is inaccessible, or metadata is unavailable, stop and report.

Choose the source branch from the base repository: the PR base if it is `master` or `main`; otherwise the repository's default branch if it is `master` or `main`. If neither identifies the source, ask the user which `master` or `main` branch to use. Match Git remotes to the base and head repositories, including forks; if either cannot be identified or fetched, stop rather than guessing `origin`.

Anchor `<pr-intent>` — why the PR exists: title, body, and, when the PR references issues or cards, a ticket brief (see Ticket Brief) scoped to intended behavior and constraints. No usable brief → fall back to title/body and note the gap. Step 3 resolves conflicts against it.

## 2. Prepare the PR branch

Inspect the worktree, index, and Git operation state before switching or merging. If there are uncommitted or untracked changes, an existing merge/rebase, or a PR branch checked out in another worktree, stop and preserve them.

Fetch the PR head from its head repository and record its commit → `<pr-head-commit>`. Check out the PR branch at that commit — an existing local `<pr-branch>` counts only when its upstream is the head repository's `refs/heads/<pr-branch>`; a same-named branch without that upstream is a collision → check out a distinct ref at `<pr-head-commit>`, never modify it. An existing local branch behind → sync it yourself: fast-forward to `<pr-head-commit>` — behind alone never blocks. Ahead of it → continue; the push publishes the local commits with the merge — flag them in the report. Diverged → `git rebase <pr-head-commit>` only when the replay range is linear — merge commits can carry edits a clean replay silently drops → a merge in the range → stop; a clean replay — already-upstream commits drop on their own — → continue, flag the replayed commits in the report; conflict → `git rebase --abort`, stop and report. Verify the checked-out branch and that `<pr-head-commit>` is an ancestor of it; record the checked-out tip → `<aligned-tip>`. Never reset, overwrite, or discard local work.

Fetch the chosen source branch from the base repository and record its fetched commit → `<source-commit>`. Already an ancestor of `<pr-head-commit>` → report that the branch is current and stop. Already an ancestor of `<aligned-tip>` → the merge exists locally unpublished — skip to Step 4's approval gate; Step 5 then pushes the tip with nothing to complete.

## 3. Merge and resolve

Merge the recorded source commit into the PR branch with `git merge --no-ff --no-commit <source-commit>`, leaving the merge pending for review and approval. Investigate each conflict using the merge base, both sides, and the surrounding behavior. Preserve the intent of both changes — judge the PR side against `<pr-intent>`; do not blindly select one side. Resolve and stage each affected path, including deletions and renames. The base removing or replacing behavior `<pr-intent>` assumes is premise drift → a product decision per below, never a silent resolution.

If a conflict requires a product decision or cannot be resolved safely, leave the merge pending and ask the user. Do not abort the merge or discard changes without explicit approval. If a merge error is unrelated to conflicts, stop and report.

## 4. Verify and commit

Confirm `git ls-files -u` is empty, inspect the staged merge for conflict markers and unintended changes, and run relevant checks for the resolved files. Report any checks that fail or cannot run; leave the merge pending until failures are understood.

Present the source and PR branches, the verified push destination `<head-repository>:<pr-branch>` (fork heads included), resolved paths, and verification results. Ask (header `Approval`): "Ready to commit and push to `<head-repository>:<pr-branch>`?" Options: `Go Ahead` → `approved`; `Needs Review` → `concerns` (requires feedback). `concerns` → apply the feedback, re-verify, re-present.

## 5. Commit and push

On `approved`: a pending merge → complete it with the repository's commit convention and verify its parents are `<aligned-tip>` and `<source-commit>`; a merge already in the tip → nothing to complete. Re-fetch the PR head first; if it changed since preparation, stop and report rather than overwriting it. Push normally to the verified head repository and PR branch, never force-push. Report the merge commit — or the pre-existing tip — and the pushed ref.

## Ticket Brief

When a step calls for a ticket brief, reuse session results first — a prior brief covering the same references and scope, or the referenced tickets' content already loaded at the depth the brief requires (comments and attachments included; a lighter read does not qualify) → derive the scoped answers without delegating. Otherwise delegate to one isolated, read-only subagent: pass only the ticket references, the scope, `<project-root>`, and these rules — the main session receives only the returned brief. Worker: read `<project-root>/docs/issue-trackers.md` and use its read operations; load each in-scope ticket's body, checklists, full comment thread, and accessible attachments (inspect contents, not filenames); follow only links bearing on the scope; dedupe by tracker and ID. Return a compact cited brief — the scoped answers, confirmed decisions distinguished from proposals and superseded ones, blockers and evidence gaps; cite comment permalinks or attachment name plus location; omit raw threads and attachment contents; mark the brief partial when missing evidence could change it. Never post or modify tracker state. Worker or spec unavailable → report the gap without fetching inline.
