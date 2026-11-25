---
name: pr-fix-ci
description: "Activate when a pull request has failing CI checks that require code fixes."
---

## Required `<project-root>/docs` reads

Shell-read before fixing failing CI checks; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`

Fix failing CI checks on a PR: read logs, fix code, push, verify. Tools: git host (`<project-root>/docs/git-hosts.md`). Check source: `gh pr checks` / `gh pr view --json statusCheckRollup`.

Modes: `review` (default) or `auto` — `auto` skips the Step 10 approval gate. `<pr-ref>` is the only authoritative source for the target PR — never infer from the current branch.

### 1. Interpret Arguments
`auto` → `<execution-mode>` = `auto`; extra guidance → `<additional-context>`; otherwise → `review`. Resolve `<pr-ref>` from the explicit PR number/URL, else the unambiguous PR targeted by the session's current work. Missing or ambiguous → ask; never pick an unrelated PR. Load current PR and check state even when the reference came from the session.

### 2. Load PR Context
Verify git host auth (`git-hosts.md`). `git rev-parse --show-toplevel` → `<repo-path>` (use as `cwd`). View PR via git host view PR (includes checks) or spec CLI fallback. Extract `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`. `git branch --show-current` → `<current-branch>`.

### 3. Align Local Branch
`<pr-branch>` unavailable → STOP. Inspect the worktree and index; uncommitted or untracked changes → STOP and ask the user to preserve them before continuing. Fetch the PR head and record its commit → `<pr-head-commit>`. Check out `<pr-branch>` — `git switch <pr-branch>` when it exists and tracks the head repository's `refs/heads/<pr-branch>`; any other same-named branch is a collision → `git switch -c` a distinct branch at `<pr-head-commit>`, never modify it; only when no same-named branch exists → git host checkout PR or `gh pr checkout <pr-number>` (checkout tools fast-forward only and fail on a diverged branch) → `<active-branch>`; fails → STOP; match a Git remote to the head repository — fork checkouts may add it → `<head-remote>`. Behind `<pr-head-commit>` → sync it yourself per `git-hosts.md`: fast-forward to `<pr-head-commit>` — behind alone never blocks. Ahead (unpushed local commits on top of `<pr-head-commit>`) → continue; the flow's push publishes them — flag them in the report. Diverged → `git rebase <pr-head-commit>` only when the replay range is linear — merge commits can carry edits a clean replay silently drops → a merge in the range → STOP; a clean replay — already-upstream commits drop on their own — leaves the branch containing `<pr-head-commit>` → continue, flag the replayed commits in the report; conflict → `git rebase --abort`, STOP and report. No code edits until HEAD is on `<active-branch>` and `<pr-head-commit>` is an ancestor of it — `<pr-branch>` stays the remote destination, not a required local name.

### 4. Identify Failing Checks
From `statusCheckRollup` (or `gh pr checks`), collect checks in `FAILURE`, `ERROR`, `CANCELLED`, `TIMED_OUT` → `<check>` with `<name>`, `<state>`, `<run-id>` (Actions only), `<url>`. Pending/in-progress → wait and re-poll once (transient). None failing → output "No failing checks", stop.

### 5. Read Failure Logs
Per `<check>`: Actions run → `gh run view <run-id> --log-failed` (fallback `--log` if empty). Non-Actions → open `<url>` and read rendered output via `webfetch` if reachable, else record `<url>` for the user. Extract `<error-lines>` (first error/stack/assertion), `<failing-file>`, `<failing-test>`, `<failing-command>`.

### 6. Triage Failures
Classify each `<check>`:
- `code` — compile error, test failure, or lint issue, including unknown PR attribution → provisional until the premise check; fix in-scope failures in Step 8.
- `flaky` — transient (network timeout, race, known-flaky test) → re-run, don't fix code.
- `infra` — environment/runner (missing secret, service down, quota) → report; re-run only if user confirms.
- `required-status` — required check missing/pending (e.g. no recent review) → out of scope; report.

Record `<classification>`; `flaky`/`infra`/`required-status` → `<not-fixing>` with reason. All failures non-`code` → summary + stop (suggest re-runs where applicable).

For remaining `code` failures, anchor `<pr-intent>` — why the PR exists: title, body, commit messages, and a ticket brief (see Ticket Brief) for all issue/card references in the body, commits, or branch name, scoped to intended behavior. For each scope-relevant target PR decision reference returned by the brief, reuse loaded feedback or retrieve only that decision and the context needed to establish its status via the git host (`git-hosts.md`); unavailable or ambiguous → note the gap. No usable brief → fall back to PR title/body plus any verified decision; note the gap.

Premise check — a `code` failure with unknown PR attribution, in code the PR doesn't touch, or showing `<pr-intent>`'s assumed behavior no longer holds → verify against `<base-branch>`; pre-existing or drifted → `<not-fixing>` with reason. Don't widen scope silently. No in-scope `code` failures remain → summary + stop.

### 7. Plan Fix
Fast path: every `code` failure straightforward, no cross-check dependencies → `<implementation-plan>` = `trivial` — on this path partition `code` failures into `<fix-groups>` (shared file lands in one group) and mark `parallel-eligible` per the criteria below before skipping to Step 8 when dispatch is viable: three or more groups or the user's explicit request. Otherwise ordered fix list (compile/build → test → lint; then shared file, then check); sequential thinking merges failures sharing one root cause → `<dependencies>`; partition `code` failures into `<fix-groups>` (shared root cause or shared file lands in one group) and mark a group `parallel-eligible` when its files are disjoint from the other groups, it has no cross-group dependencies, and it runs no commands writing shared mutable state such as lockfiles or build caches.

### 8. Implement Fix
On `<active-branch>` — focused, minimal, existing patterns. `parallel-eligible` `<fix-groups>` may be implemented in parallel — dispatch one subagent per group, each editing only its assigned files — only with three or more groups or the user's explicit request to delegate, and only when a suitable subagent profile exists; shared-cause or doubtful-independence failures stay serial. Collect each group's changed files into `<fix-files>`, verifying they stayed within its boundaries; a failed group's changes are discarded or reset before validating or pushing — never mix successful and failed group edits; its checks are retried serially in this session, and checks still failing are recorded unresolved in `<validation-results>` and surfaced at the approval gate or STOP report — never counted as fixed. Local validation, commit, and push stay serial. Root cause, not symptom: never disable a failing test — fix what it asserts unless the assertion is wrong (then fix it with justification) → `<changes-count>` and `<fix-files>`.

### 9. Validate Locally
Reproduce each fixed `<check>`'s `<failing-command>` locally. Set `<validation-passing>` to `no` if any local check fails, including a failed serial retry from Step 8; `unverified` if any check could not run or remains unresolved without a confirmed local result; `yes` only if all pass. Record the details in `<validation-results>` and list unresolved checks at the gate or STOP report.

### 10. Approval Gate (mandatory in `review` mode)
Same gate as `pr-fix` Step 9 — `review`/`auto` semantics, `Go Ahead`/`Needs Review` loop (refine → re-validate → re-present until `approved`; refinement that changes which files a fix touches re-derives `<fix-groups>` first). Present: failing checks fixed, file count, local validation, `<not-fixing>` items (flaky/infra with suggested re-runs). `<changes-count>` = 0 → surface no-change state.

### 11. Commit and Push (after `approved` or `auto`)
Inspect the worktree and index again, including changes within `<fix-files>`. Unexpected changes → STOP and preserve them. Stage and commit only `<fix-files>` with explicit pathspecs (message convention per `commit-and-push`), then `git push <head-remote> <active-branch>:<pr-branch>`. Rejected non-fast-forward — the head moved since `<pr-head-commit>` → re-fetch; `git rebase --onto <new-head> <pr-head-commit>` — replays only commits added on top of the recorded head, so replaced PR commits never return; linear range only → a merge → STOP; clean → re-run local validation on the rebased tree, retry once, re-record `<pr-head-commit>`; conflict → `git rebase --abort`, STOP — never force-push. Never use a generic message like "fix ci". Do not poll CI or loop back — session ends after push.

## Ticket Brief

Reuse a current session brief, summary, or ticket content covering the references and scoped answers, with sufficient evidence and decision status for that scope. Otherwise delegate one isolated, read-only subagent: pass the ticket references, scope, `<project-root>`, these rules, and the target PR reference when known; the parent receives only the brief.

Worker: read `<project-root>/docs/issue-trackers.md` and use its read operations. Start with each ticket's title, body, checklists, and requested metadata; dedupe by tracker and ID. When scope includes intended, expected, or current behavior or constraints, check the latest page of ticket comments for updates. Read older comments, attachment contents, or linked tickets only when needed to answer the scope or resolve conflicting or unclear decisions. These scoped rules override the spec's general attachment and nested-link traversal. Do not treat the target PR as a ticket or fetch its comments or reviews; return scope-relevant target PR decision references as evidence gaps for the parent. Follow other PR links only when needed for the scope, and retrieve specific relevant evidence rather than full comment or review history. If a provider bundles extra content, inspect only what the scope requires.

Return a compact cited brief with scoped answers, decision status, blockers, and evidence gaps; cite ticket, comment permalink, or attachment name and location as appropriate. Omit raw threads and attachment contents. Distinguish confirmed decisions from proposals and superseded decisions when evidenced; mark answers partial when evidence indicates uninspected or inaccessible material could change them. Stop when the scoped answers are supported; never post or modify tracker state. Worker or spec unavailable → report the gap without fetching inline.

## Output
Summary of fixed checks, local validation, pushed state — one line per fix at the gate, e.g. "Fixed N failing checks across M files" + validation + "pushed to `<active-branch>`". Never claim CI green — unverified.
