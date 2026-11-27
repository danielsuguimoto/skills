# Standalone shipping

Use the matching section only when its companion skill is unavailable. Keep the main skill's scope, approvals, and `<excluded>` gates.

## Branch

Categories: `feature`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `build`, `ci`. Derive a 2–4 word kebab-case slug from the ticket title or `<change-summary>`. A current work branch with one of these prefixes, `feat`, `bugfix`, or `hotfix` → stop rather than branch again. Create `<category>/<slug>` at the current checkout without tracking the base (`git-hosts.md`; fallback `git checkout --no-track -b <branch>`); verify no upstream. Name collision → retry once with `-2`. Failure → stop.

## Commit and push

Stage only `<included>` paths and commit only those paths (`git-hosts.md`; fallback `git add -- <included>` then `git commit -m "<message>" -- <included>`). Use `<prefix>: <message>`, lowercase, under 50 characters, no body; prefixes: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `style`. No included changes → skip the commit. Record the destination remote tip before pushing; publish via `git-hosts.md` or `git push`, using `git push -u origin <branch>` when no upstream exists. A rejected non-fast-forward → fetch the exact destination; with no recorded remote tip, stop rather than guess a replay range. Otherwise replay only local commits above the recorded tip with `git rebase --onto <new-tip> <remote-tip>`, linear ranges only. Merge commits or a conflict → stop; abort a conflicting rebase. Retry once after a clean replay; never force-push. Other failures → stop. Return HEAD as `<hash>` and the actual remote and full branch ref as `<push-target>`.

## Pull request

Before branching or publishing, verify that `git-hosts.md` provides PR creation and lookup, or that this is a GitHub repository with an authenticated `gh` CLI. Neither available → stop and report. Detect the default base unless `<base>` was supplied; fetch it and compare `origin/<base>...HEAD`. Stop for uncommitted changes outside `<excluded>`, staged exclusions, a base-branch checkout, or an empty diff. Probe every repository PR-template location before composing; retain the selected template's headings and checkboxes. If none exists, use Ticket, Issue, Description, and Checklist headings. Include the problem and the actual changes, with `SKIPPED` for no ticket. Generate `<pr-title>`: lowercase `<prefix>: <message>`, under 50 characters.

Verify the published head, destination remote ref, and upstream all match HEAD; otherwise push with upstream, never force-push. Create the PR via `git-hosts.md` with `<pr-title>`, resolved base, current head, body, and assignee; if it already exists, load it. GitHub CLI fallback: `gh pr create --title "<pr-title>" --base <base> --head <branch> --assignee @me --body "<pr-body>"`; pass the actual body inline with shell quoting that preserves newlines. If it exists, use `gh pr view <branch> --json url,number`. Verify the rendered body via the host or `gh pr view <branch> --json body`; correct it if needed. Return its URL.

For each verified, in-scope ticket reference in the selected ticket or PR body, post `PR: <pr-url>` via `issue-trackers.md`; ask before linking an ambiguous or unverified target. No ticket and no issue references → skip. Report any failed or unsupported link.
