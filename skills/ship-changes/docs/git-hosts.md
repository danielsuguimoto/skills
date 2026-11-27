# Git host fallback

Use only for operations missing from `<project-root>/docs/git-hosts.md`. The calling workflow owns approvals, scope, and safety. Git operations run on the host shell.

| Operation | Git / GitHub CLI fallback |
|---|---|
| Scan | `git branch --show-current`; `git status --short --untracked-files=all`; `git diff --stat`; `git diff --cached --stat`; `git log @{u}..HEAD --oneline` |
| Branch | `git checkout --no-track -b <branch>` |
| Commit | `git add -- <included>`; `git commit -m "<message>" -- <included>` |
| Push | `git push <head-remote> <local-branch>:<verified-head-ref>`; add `-u` only when no upstream exists |
| Replay | `git rebase --onto <new-tip> <remote-tip>`; conflict → `git rebase --abort` |
| Host identity | `gh auth status --hostname <base-host>`; `gh repo view --json nameWithOwner,defaultBranchRef`; inspect the base and push remote URLs and repository metadata with `gh repo view <host>/<owner>/<repo> --json nameWithOwner,owner,parent` |
| Create PR | `gh pr create --repo <base-host>/<base-owner>/<base-repo> --title "<pr-title>" --base <base> --head <head> --assignee @me --body "<pr-body>"` |
| Find PR | `gh pr list --repo <base-host>/<base-owner>/<base-repo> --head <branch> --json number,url,headRepository,headRepositoryOwner,headRefName` |
| Verify body | `gh pr view <number> --repo <base-host>/<base-owner>/<base-repo> --json body` |
| Edit body | `gh pr edit <number> --repo <base-host>/<base-owner>/<base-repo> --body "<pr-body>"` |

For a GitHub cross-fork head, `<head>` for creation is `<verified-user-owner>:<branch>`; same repository uses `<branch>`. Base and head must be on the same verified host. The CLI does not support organization-owned cross-fork heads: stop before pushing if that is the only available PR-creation path. Listing accepts only the bare branch, so filter its results by the verified head owner, repository, and ref. Pin the base host and repository for every PR operation. A missing, ambiguous, or unrepresentable target → stop. Pass bodies inline with shell quoting that preserves newlines. If a write result is uncertain, look up the PR before retrying.
