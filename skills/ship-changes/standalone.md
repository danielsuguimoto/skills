# Standalone shipping

Use the sections selected by the main skill. Keep its scope, approvals, and `<excluded>` gates. Use the project `git-hosts.md` operations when configured; for missing operations read the [bundled fallback spec](docs/git-hosts.md).

## Destination

Before publishing with the standalone publisher, resolve the actual push remote, its host/repository, and full destination branch ref from explicit input or the branch's configured push destination. A new branch without an upstream uses the configured push remote, else `origin`, with the new branch name as its destination. Verify the remote's push URL. A branch not yet created gets its destination ref after creation and before publication. Missing, ambiguous, or unsupported destination → stop before publication. Keep the verified remote and full ref as `<push-destination>` and pass them to the publisher.

## Branch

Categories: `feature`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `build`, `ci`. Derive a 2–4 word kebab-case slug from the ticket title or `<change-summary>`. A current work branch with one of these prefixes, `feat`, `bugfix`, or `hotfix` → stop rather than branch again. Create `<category>/<slug>` at the current checkout without tracking the base; verify no upstream. Name collision → retry once with `-2`. Failure → stop.

## Commit and push

Stage only `<included>` paths and commit only those paths. Use `<prefix>: <message>`, lowercase, under 50 characters, no body; prefixes: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `style`. No included changes → skip the commit. Record the destination remote tip before pushing; publish to `<push-destination>` using an operation that accepts its exact remote and full ref. If a configured operation cannot target both, use the bundled fallback. Set upstream only when absent; an existing different upstream → stop before publishing rather than change it. A rejected non-fast-forward → fetch the exact destination; with no recorded remote tip, stop rather than guess a replay range. Otherwise replay only local commits above the recorded tip, linear ranges only. Merge commits or a conflict → stop; abort a conflicting rebase. Retry once after a clean replay; never force-push. Other failures → stop. Return HEAD as `<hash>` and the actual remote and full branch ref as `<push-target>`.

## Pull request

Before branching or publishing, verify that the project spec provides PR creation and lookup or the bundled fallback is usable. Resolve the intended base host/repository and a remote for it from explicit user input or unambiguous session and repository context; ambiguous → ask, unavailable → stop. Resolve the push remote and repository per Destination; finalize a new branch's ref after creation and before publication. Map a cross-repository head only when the host supports that owner and branch; otherwise stop before publishing. Store the base host/repository, base remote, and head host/repository/ref as the preflight target.

Detect the default base unless `<base>` was supplied; fetch it from the verified base remote and compare `<base-remote>/<base>...HEAD`. Stop for uncommitted changes outside `<excluded>`, staged exclusions, a base-branch checkout, or an empty diff. Probe every repository PR-template location before composing; retain the selected template's headings and checkboxes. If none exists, use Ticket, Issue, Description, and Checklist headings. Include the problem and the actual changes, with `SKIPPED` for no ticket. Generate `<pr-title>`: lowercase `<prefix>: <message>`, under 50 characters.

Verify the actual `<push-target>` matches the preflight target and its remote ref and upstream match HEAD; mismatch → stop rather than publish to an unverified destination. Create or load the PR in the verified base repository with `<pr-title>`, resolved base, verified head repository/ref, body, and assignee. Pin that repository for lookup and body verification; correct the rendered body if needed. Return its URL.

For each verified, in-scope ticket reference in the selected ticket or PR body, post `PR: <pr-url>` via `issue-trackers.md`; ask before linking an ambiguous or unverified target. No ticket and no issue references → skip. Report any failed or unsupported link.
