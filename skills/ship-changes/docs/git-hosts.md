# Git host fallback

Use only for operations missing from `<project-root>/docs/git-hosts.md`. The calling workflow owns approvals, scope, and safety. Git operations run on the host shell.

| Operation | Git / GitHub CLI fallback |
|---|---|
| Scan | `git branch --show-current`; `git status --short`; `git diff --stat`; `git diff --cached --stat`; `git log @{u}..HEAD --oneline` |
| Branch | `git checkout --no-track -b <branch>` |
| Commit | `git add -- <included>`; `git commit -m "<message>" -- <included>` |
| Push | `git push -u <head-remote> <branch>:refs/heads/<branch>` to the preflighted destination |
| Replay | `git rebase --onto <new-tip> <remote-tip>`; conflict → `git rebase --abort` |
| Host identity | `gh auth status`; `gh repo view --json nameWithOwner,defaultBranchRef`; inspect the base and push remote URLs and repository metadata with `gh repo view <owner>/<repo> --json nameWithOwner,owner,parent` |
| Create PR | `gh pr create --repo <base-owner>/<base-repo> --title "<pr-title>" --base <base> --head <head> --assignee @me --body "<pr-body>"` |
| Find PR | `gh pr list --repo <base-owner>/<base-repo> --head <branch> --json number,url,headRepository,headRepositoryOwner,headRefName` |
| Verify body | `gh pr view <number> --repo <base-owner>/<base-repo> --json body` |
| Edit body | `gh pr edit <number> --repo <base-owner>/<base-repo> --body "<pr-body>"` |

For a GitHub cross-fork head, `<head>` for creation is `<verified-user-owner>:<branch>`; same repository uses `<branch>`. The CLI does not support organization-owned cross-fork heads: stop before pushing if that is the only available PR-creation path. Listing accepts only the bare branch, so filter its results by the verified head owner, repository, and ref. Pin the base repository for every PR operation. A missing, ambiguous, or unrepresentable target → stop. Pass bodies inline with shell quoting that preserves newlines. If a write result is uncertain, look up the PR before retrying.
