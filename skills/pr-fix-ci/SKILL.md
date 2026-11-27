---
name: pr-fix-ci
description: "Fix code that causes PR checks to fail."
---

## Required `<project-root>/docs` reads

Shell-read before fixing failing CI checks (reuse session reads): `<project-root>/docs/git-hosts.md`. If a spec is missing, report the gap and use a documented fallback.

Fix failing CI checks on a PR: read logs, fix code, push, verify. Tools: git host (`<project-root>/docs/git-hosts.md`). Check source: `gh pr checks` / `gh pr view --json statusCheckRollup`.

Modes: `review` (default) or `auto` — `auto` skips the Step 10 approval gate. `<pr-ref>` is the only authoritative source for the target PR — never infer from the current branch.

### 1. Interpret Arguments
`auto` → `<execution-mode>` = `auto`; extra guidance → `<additional-context>`; otherwise → `review`. Resolve `<pr-ref>` from the explicit PR number/URL, else the unambiguous PR targeted by the session's current work. Missing or ambiguous → ask; never pick an unrelated PR. Load current PR and check state even when the reference came from the session.

### 2. Load PR Context
Verify git host auth (`git-hosts.md`). `git rev-parse --show-toplevel` → `<repo-path>` (use as `cwd`). View the current PR, including its `statusCheckRollup`, via git host view PR or the spec CLI fallback. Extract `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`. `git branch --show-current` → `<current-branch>`.

### 3. Identify Failing Checks
Reuse Step 2's check rollup; if the host omitted it, load checks via `git-hosts.md` before deciding. Collect checks in `FAILURE`, `ERROR`, `CANCELLED`, `TIMED_OUT` → `<check>` with `<name>`, `<state>`, `<run-id>` (Actions only), `<url>`. If none failed, report pending checks if any, otherwise report no failing checks; stop without waiting.

### 4. Read Failure Logs
Per `<check>`: Actions run → `gh run view <run-id> --log-failed` (fallback `--log` if empty). Non-Actions → open `<url>` and read rendered output via `webfetch` if reachable, else record `<url>` for the user. Extract `<error-lines>` (first error/stack/assertion), `<failing-file>`, `<failing-test>`, `<failing-command>`.

### 5. Triage Failures
The PR body, commits, or branch name references an issue/card AND every `<check>` appears `flaky`, `infra`, or `required-status` per the definitions below → compare failure evidence with the PR title/body, commits, and changed paths via the git host (`git-hosts.md`). Only when that evidence rules out a PR-related cause for every failure → classify and record each as below; summary + stop (suggest re-runs where applicable). A possible connection or missing evidence → continue; known flakiness alone does not establish independence. Until checkout, compare PR and base evidence via the git host or fetched refs, not the current working tree.

Anchor `<pr-intent>` — why the PR exists: title, body, commit messages, and a ticket brief (see Ticket Brief) for all issue/card references in the body, commits, or branch name, scoped to intended behavior. For each scope-relevant target PR decision reference returned by the brief, reuse loaded feedback or retrieve only that decision and the context needed to establish its status via the git host (`git-hosts.md`); unavailable or ambiguous → note the gap. No usable brief → fall back to PR title/body plus any verified decision; note the gap.

When the PR references issues/cards and `<pr-intent>` or PR changes target a failure, investigate the PR-related cause before classifying it as `flaky`.

Classify each `<check>`:
- `code` — caused by the PR's code (compile error, test failure, lint) → fix in Step 8.
- `flaky` — transient (network timeout, race, known-flaky test) → re-run, don't fix code.
- `infra` — environment/runner (missing secret, service down, quota) → report; re-run only if user confirms.
- `required-status` — required check missing/pending (e.g. no recent review) → out of scope; report.
- Premise check — a `code` failure in code the PR doesn't touch, or one showing `<pr-intent>`'s assumed behavior no longer holds → verify against `<base-branch>`; pre-existing or drifted → `<not-fixing>` with reason. Don't widen scope silently.

Record `<classification>`; `flaky`/`infra`/`required-status` → `<not-fixing>` with reason. All failures non-`code` → summary + stop (suggest re-runs where applicable).

### 6. Align Local Branch
Use a composite alignment operation only when it guarantees the branch, replay, and preservation rules below; otherwise treat it as absent.
`<pr-branch>` unavailable → STOP. Inspect the worktree and index; uncommitted or untracked changes → STOP and ask the user to preserve them before continuing. Fetch the PR head and record its commit → `<pr-head-commit>`. Align via `Align PR branch for editing` in `git-hosts.md` → `<active-branch>`; fails → STOP. Spec lacks the operation → the fallback: `git switch <pr-branch>` when it exists and tracks the head repository's `refs/heads/<pr-branch>`; any other same-named branch is a collision → `git switch -c` a distinct branch at `<pr-head-commit>`, never modify it; no same-named branch → git host checkout PR or `gh pr checkout <pr-number>` (fast-forward only, fails on a diverged branch) → `<active-branch>`. After checkout, match a Git remote to the head repository — fork checkouts may add it → `<head-remote>`; no match → STOP. Behind → fast-forward to `<pr-head-commit>` — behind alone never blocks. Ahead → continue; the flow's push publishes them — flag them in the report. Diverged → `git rebase <pr-head-commit>`, linear range only — a merge in the range → STOP; a clean replay (already-upstream commits drop on their own) → continue, flag the replayed commits in the report; conflict → `git rebase --abort`, STOP and report. No code edits until HEAD is on `<active-branch>` and `<pr-head-commit>` is an ancestor of it — `<pr-branch>` stays the remote destination, not a required local name.

### 7. Plan Fix
Fast path: every `code` failure straightforward, no cross-check dependencies → `<implementation-plan>` = `trivial` — on this path partition `code` failures into `<fix-groups>` (shared file lands in one group) and mark `parallel-eligible` per the criteria below before skipping to Step 8 when dispatch is viable: three or more groups or the user's explicit request. Otherwise ordered fix list (compile/build → test → lint; then shared file, then check); sequential thinking merges failures sharing one root cause → `<dependencies>`; partition `code` failures into `<fix-groups>` (shared root cause or shared file lands in one group) and mark a group `parallel-eligible` when its files are disjoint from the other groups, it has no cross-group dependencies, and it runs no commands writing shared mutable state such as lockfiles or build caches.

### 8. Implement Fix
On `<active-branch>` — focused, minimal, existing patterns. `parallel-eligible` `<fix-groups>` may be implemented in parallel — dispatch one subagent per group, each editing only its assigned files — only with three or more groups or the user's explicit request to delegate, and only when a suitable subagent profile exists; shared-cause or doubtful-independence failures stay serial. Collect each group's changed files into `<fix-files>`, verifying they stayed within its boundaries; a failed group's changes are discarded or reset before validating or pushing — never mix successful and failed group edits; its checks are retried serially in this session, and checks still failing are recorded unresolved in `<validation-results>` and surfaced at the approval gate or STOP report — never counted as fixed. Local validation, commit, and push stay serial. Root cause, not symptom: never disable a failing test — fix what it asserts unless the assertion is wrong (then fix it with justification) → `<changes-count>` and `<fix-files>`.

### 9. Validate Locally
Reproduce each fixed `<check>`'s `<failing-command>` locally. Set `<validation-passing>` to `no` if any local check fails, including a failed serial retry from Step 8; `unverified` if any check could not run or remains unresolved without a confirmed local result; `yes` only if all pass. Record the details in `<validation-results>` and list unresolved checks at the gate or STOP report.

### 10. Approval Gate (mandatory in `review` mode)
Stop on failed validation or no fixes. Show the PR, fixed checks, file count, local validation (including unverified checks), deferred failures, and existing commits the push would publish. In `review` mode ask once to commit and push this batch: `Go Ahead` authorizes both; `Needs Review` means refine, regroup if files changed, revalidate, and ask again. An unanswered or declined gate stops publication. Explicit `auto` authorizes commit and push for this batch without asking. Honor tool-enforced permissions; no second confirmation for the approved scope.

### 11. Commit and Push (after `approved` or `auto`)
Use a composite push-recovery operation only when it guarantees the replay, revalidation, and retry rules below; otherwise treat it as absent.
Inspect the worktree and index again, including changes within `<fix-files>`. Unexpected changes → STOP and preserve them. If the target or batch changes materially after approval, return to Step 10 in `review` mode, even if the run started in `auto`. Stage and commit only `<fix-files>` with explicit pathspecs (message convention per `commit-and-push`), then `git push <head-remote> <active-branch>:<pr-branch>`. Rejected non-fast-forward — the head moved since `<pr-head-commit>` → `Recover rejected PR push` in `git-hosts.md`; spec lacks the operation → re-fetch; `git rebase --onto <new-head> <pr-head-commit>` — replays only commits added on top of the recorded head, so replaced PR commits never return; linear range only → a merge → STOP; clean → revalidate on the rebased tree; retry once only if validation passes and the batch is unchanged, then record `<pr-head-commit>`; conflict → `git rebase --abort`, STOP — never force-push. Never use a generic message like "fix ci". After a successful push, report and stop without polling CI.

## Ticket Brief

Reuse current, scoped evidence; otherwise use `issue-trackers.md` to dedupe by tracker/ID, read only scoped fields, and read the latest comment page for behavior or constraints. Follow older material or other links only to settle a gap; for the target PR return decision references without fetching feedback. Return cited answers, decision status (confirmed/proposed/superseded), gaps and blockers; mark partial answers and leave tracker state unchanged. Read inline, or delegate one bounded read-only brief when it saves distracting context or the user requests it; pass the refs, scope, `<project-root>` and target PR. Worker unavailable → inline; completed lookup blocker → do not retry. Missing spec → report the gap.

## Output
Summary of fixed checks, local validation, pushed state — one line per fix at the gate, e.g. "Fixed N failing checks across M files" + validation + "pushed to `<active-branch>`". Never claim CI green — unverified.
