# Standalone shipping

Use the matching section only when its companion skill is unavailable. Keep the main skill's scope, approvals, and `<excluded>` gates.

## Branch

Categories: `feature`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `build`, `ci`. Derive a 2–4 word kebab-case slug from the ticket title or `<change-summary>`. A current work branch with one of these prefixes, `feat`, `bugfix`, or `hotfix` → stop rather than branch again. Create `<category>/<slug>` at the current checkout without tracking the base (`git-hosts.md`); verify no upstream. Name collision → retry once with `-2`. Failure → stop.

## Commit and push

Stage only `<included>` paths and commit only those paths. Use `<prefix>: <message>`, lowercase, under 50 characters, no body; prefixes: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `style`. No included changes → skip the commit. Record the destination remote tip before pushing; publish via `git-hosts.md`, setting the new branch's upstream when needed. A rejected non-fast-forward → fetch the exact destination; replay only local commits above the recorded tip with `git rebase --onto <new-tip> <remote-tip>`, linear ranges only. Merge commits or a conflict → stop; abort a conflicting rebase. Retry once after a clean replay; never force-push. Other failures → stop. Return HEAD as `<hash>` and the actual remote and full branch ref as `<push-target>`.

## Pull request

Detect the default base unless `<base>` was supplied; fetch it and compare `origin/<base>...HEAD`. Stop for uncommitted changes outside `<excluded>`, staged exclusions, a base-branch checkout, or an empty diff. Probe every repository PR-template location before composing; retain the selected template's headings and checkboxes. If none exists, use Ticket, Issue, Description, and Checklist headings. Include the problem and the actual changes, with `SKIPPED` for no ticket. Title: lowercase `<prefix>: <message>`, under 50 characters.

Verify the published head, destination remote ref, and upstream all match HEAD; otherwise push with upstream, never force-push. Create the PR via `git-hosts.md` with the resolved base, current head, body, and assignee; if it already exists, load it. Verify the rendered body and correct it if needed. Return its URL.

For each verified, in-scope ticket reference in the selected ticket or PR body, post `PR: <pr-url>` via `issue-trackers.md`; ask before linking an ambiguous or unverified target. No ticket and no issue references → skip. Report any failed or unsupported link.
