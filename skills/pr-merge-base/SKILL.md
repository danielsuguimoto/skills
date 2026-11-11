---
name: pr-merge-base
description: "Use when merging the latest master or main branch into a pull request branch, including conflict resolution."
---

## Required `<project-root>/docs` reads

Shell-read before loading PR metadata: `<project-root>/docs/git-hosts.md`. If missing or incomplete, note the gap; use a supported git host or CLI fallback.

## 1. Identify the PR and branches

Require a PR number or URL argument; if absent, ask for it. Use that PR as the sole target, not the current branch. Load its state, base repository and branch, head repository and branch, and URL through the git host. If the PR is closed, its head branch is missing, a repository is inaccessible, or metadata is unavailable, stop and report.

Choose the source branch from the base repository: the PR base if it is `master` or `main`; otherwise the repository's default branch if it is `master` or `main`. If neither identifies the source, ask the user which `master` or `main` branch to use. Match Git remotes to the base and head repositories, including forks; if either cannot be identified or fetched, stop rather than guessing `origin`.

## 2. Prepare the PR branch

Inspect the worktree, index, and Git operation state before switching or merging. If there are uncommitted or untracked changes, an existing merge/rebase, or a PR branch checked out in another worktree, stop and preserve them.

Fetch the PR head from its head repository and record its commit. Check out the PR branch at that commit. If an existing local branch is behind, fast-forward it only; if it has local commits or has diverged, stop and report. Verify the checked-out branch and commit against the PR head before continuing. Never reset, overwrite, or discard local work.

Fetch the chosen source branch from the base repository and record its fetched commit. If that commit is already an ancestor of the PR head, report that the branch is current and stop.

## 3. Merge and resolve

Merge the recorded source commit into the PR branch with `git merge --no-ff --no-commit <source-commit>`, leaving the merge pending for review and approval. Investigate each conflict using the merge base, both sides, and the surrounding behavior. Preserve the intent of both changes; do not blindly select one side. Resolve and stage each affected path, including deletions and renames.

If a conflict requires a product decision or cannot be resolved safely, leave the merge pending and ask the user. Do not abort the merge or discard changes without explicit approval. If a merge error is unrelated to conflicts, stop and report.

## 4. Verify and commit

Confirm `git ls-files -u` is empty, inspect the staged merge for conflict markers and unintended changes, and run relevant checks for the resolved files. Report any checks that fail or cannot run; leave the merge pending until failures are understood.

Present the source and PR branches, resolved paths, and verification results. Ask for explicit confirmation before creating the merge commit. On approval, complete the merge with the repository's commit convention and verify that the resulting commit has both the recorded PR head and source commit as parents.

## 5. Push only on request

The merge is local until pushed. Push only when the user explicitly requests it and confirms the destination. Re-fetch the PR head first; if it changed since preparation, stop and report rather than overwriting it. Push normally to the verified head repository and PR branch, never force-push. Report the merge commit, validation results, and whether the PR branch was pushed.
