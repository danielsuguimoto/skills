---
name: pr-merge-base
description: "Use when merging the latest master or main branch into a pull request branch, including conflict resolution."
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

Merge the recorded source commit into the PR branch with `git merge --no-ff --no-commit <source-commit>`, leaving the merge pending for review and approval. A merge error unrelated to conflicts → stop and report.

If conflicts exist, anchor `<pr-intent>` — why the PR exists: title, body, and, when the PR references issues or cards, a ticket brief (see Ticket Brief) scoped to intended behavior and constraints. For each scope-relevant target PR decision reference returned by the brief, reuse loaded feedback or retrieve only that decision and the context needed to establish its status via the git host (`git-hosts.md`); unavailable or ambiguous → note the gap. No usable brief → fall back to title/body plus any verified decision; note the gap.

Investigate each conflict using the merge base, both sides, and the surrounding behavior. Preserve the intent of both changes — judge the PR side against `<pr-intent>`; do not blindly select one side. Resolve and stage each affected path, including deletions and renames. The base removing or replacing behavior `<pr-intent>` assumes is premise drift → a product decision per below, never a silent resolution.

If a conflict requires a product decision or cannot be resolved safely, leave the merge pending and ask the user. Do not abort the merge or discard changes without explicit approval.

## 4. Verify and commit

Confirm `git ls-files -u` is empty, inspect the staged merge for conflict markers and unintended changes, and run relevant checks for the resolved files. Report any checks that fail or cannot run; leave the merge pending until failures are understood.

Present the source and PR branches, the verified push destination `<head-repository>:<pr-branch>` (fork heads included), resolved paths, and verification results. Ask (header `Approval`): "Ready to commit and push to `<head-repository>:<pr-branch>`?" Options: `Go Ahead` → `approved`; `Needs Review` → `concerns` (requires feedback). `concerns` → apply the feedback, re-verify, re-present.

## 5. Commit and push

On `approved`: a pending merge → complete it with the repository's commit convention and verify its parents are `<aligned-tip>` and `<source-commit>`; a merge already in the tip → nothing to complete. Re-fetch the PR head first; if it changed since preparation, stop and report rather than overwriting it. Push normally to the verified head repository and PR branch, never force-push. Report the merge commit — or the pre-existing tip — and the pushed ref.

## Ticket Brief

Reuse a current session brief, summary, or ticket content covering the references and scoped answers, with sufficient evidence and decision status for that scope. Otherwise delegate one isolated, read-only subagent: pass the ticket references, scope, `<project-root>`, these rules, and the target PR reference when known; the parent receives only the brief.

Worker: read `<project-root>/docs/issue-trackers.md` and use its read operations. Start with each ticket's title, body, checklists, and requested metadata; dedupe by tracker and ID. When scope includes intended, expected, or current behavior or constraints, check the latest page of ticket comments for updates. Read older comments, attachment contents, or linked tickets only when needed to answer the scope or resolve conflicting or unclear decisions. These scoped rules override the spec's general attachment and nested-link traversal. Do not treat the target PR as a ticket or fetch its comments or reviews; return scope-relevant target PR decision references as evidence gaps for the parent. Follow other PR links only when needed for the scope, and retrieve specific relevant evidence rather than full comment or review history. If a provider bundles extra content, inspect only what the scope requires.

Return a compact cited brief with scoped answers, decision status, blockers, and evidence gaps; cite ticket, comment permalink, or attachment name and location as appropriate. Omit raw threads and attachment contents. Distinguish confirmed decisions from proposals and superseded decisions when evidenced; mark answers partial when evidence indicates uninspected or inaccessible material could change them. Stop when the scoped answers are supported; never post or modify tracker state. Worker or spec unavailable → report the gap without fetching inline.
