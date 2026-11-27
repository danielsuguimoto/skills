# Standalone shipping

Use the matching section only when its companion skill is unavailable. Keep the main skill's scope, approvals, and `<excluded>` gates. Use the project `git-hosts.md` operations when configured; for missing operations read the [bundled fallback spec](docs/git-hosts.md).

## Branch

Categories: `feature`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `build`, `ci`. Derive a 2–4 word kebab-case slug from the ticket title or `<change-summary>`. A current work branch with one of these prefixes, `feat`, `bugfix`, or `hotfix` → stop rather than branch again. Create `<category>/<slug>` at the current checkout without tracking the base; verify no upstream. Name collision → retry once with `-2`. Failure → stop.

## Commit and push

Stage only `<included>` paths and commit only those paths. Use `<prefix>: <message>`, lowercase, under 50 characters, no body; prefixes: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `style`. No included changes → skip the commit. Record the destination remote tip before pushing; publish to the preflighted destination, setting upstream when needed. A rejected non-fast-forward → fetch the exact destination; with no recorded remote tip, stop rather than guess a replay range. Otherwise replay only local commits above the recorded tip, linear ranges only. Merge commits or a conflict → stop; abort a conflicting rebase. Retry once after a clean replay; never force-push. Other failures → stop. Return HEAD as `<hash>` and the actual remote and full branch ref as `<push-target>`.

## Pull request

Before branching or publishing, verify that the project spec provides PR creation and lookup or the bundled fallback is usable. Resolve the intended base repository and a remote for it from explicit user input or unambiguous session and repository context; ambiguous → ask, unavailable → stop. Resolve the planned push remote and full branch ref, including an existing branch's configured push destination; verify its repository and owner. Map a cross-repository head only when the host supports that owner and branch; otherwise stop before publishing. Store the base repository, base remote, and head repository/ref as the preflight target.

Detect the default base unless `<base>` was supplied; fetch it from the verified base remote and compare `<base-remote>/<base>...HEAD`. Stop for uncommitted changes outside `<excluded>`, staged exclusions, a base-branch checkout, or an empty diff. Probe every repository PR-template location before composing; retain the selected template's headings and checkboxes. If none exists, use Ticket, Issue, Description, and Checklist headings. Include the problem and the actual changes, with `SKIPPED` for no ticket. Generate `<pr-title>`: lowercase `<prefix>: <message>`, under 50 characters.

Verify the actual `<push-target>` matches the preflight target and its remote ref and upstream match HEAD; mismatch → stop rather than publish to an unverified destination. Create or load the PR in the verified base repository with `<pr-title>`, resolved base, verified head repository/ref, body, and assignee. Pin that repository for lookup and body verification; correct the rendered body if needed. Return its URL.

For each verified, in-scope ticket reference in the selected ticket or PR body, post `PR: <pr-url>` via `issue-trackers.md`; ask before linking an ambiguous or unverified target. No ticket and no issue references → skip. Report any failed or unsupported link.
