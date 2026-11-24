---
name: pr-fix-ci
description: "Activate when a pull request has failing CI checks that require code fixes."
---

## Required `<project-root>/docs` reads

Shell-read before fixing failing CI checks; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`

Fix failing CI checks on a PR: read logs, fix code, push, verify. Tools: git host (`<project-root>/docs/git-hosts.md`). Check source: `gh pr checks` / `gh pr view --json statusCheckRollup`.

Modes: `review` (default) or `auto` — `auto` skips the Step 10 approval gate. `<pr-ref>` is the only authoritative source for the target PR — never infer from the current branch.

### 1. Interpret Arguments
`auto` → `<execution-mode>` = `auto`; PR number/URL → `<pr-ref>`; extra guidance → `<additional-context>`; otherwise → `review`.

### 2. Load PR Context
Verify git host auth (`git-hosts.md`). `git rev-parse --show-toplevel` → `<repo-path>` (use as `cwd`). View PR via git host view PR (includes checks) or spec CLI fallback. Extract `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`. `git branch --show-current` → `<current-branch>`.

Anchor `<pr-intent>` — why the PR exists: title, body, commit messages, and a ticket brief (see Ticket Brief) for all issue/card references in the body, commits, or branch name, scoped to intended behavior. No usable brief → fall back to PR title/body; note the gap.

### 3. Align Local Branch
`<pr-branch>` unavailable → STOP. Inspect the worktree and index; uncommitted or untracked changes → STOP and ask the user to preserve them before continuing. Fetch the PR head and record its commit → `<pr-head-commit>`. Check out via git host checkout PR or `gh pr checkout <pr-number>` → `<active-branch>`; fails → STOP. Behind `<pr-head-commit>` → sync it yourself per `git-hosts.md`: fast-forward to `<pr-head-commit>` — behind alone never blocks. Diverged or unpushed local commits → no safe automatic update → STOP and report. No code edits until `<active-branch>` == `<pr-branch>` and HEAD == `<pr-head-commit>`.

### 4. Identify Failing Checks
From `statusCheckRollup` (or `gh pr checks`), collect checks in `FAILURE`, `ERROR`, `CANCELLED`, `TIMED_OUT` → `<check>` with `<name>`, `<state>`, `<run-id>` (Actions only), `<url>`. Pending/in-progress → wait and re-poll once (transient). None failing → output "No failing checks", stop.

### 5. Read Failure Logs
Per `<check>`: Actions run → `gh run view <run-id> --log-failed` (fallback `--log` if empty). Non-Actions → open `<url>` and read rendered output via `webfetch` if reachable, else record `<url>` for the user. Extract `<error-lines>` (first error/stack/assertion), `<failing-file>`, `<failing-test>`, `<failing-command>`.

### 6. Triage Failures
Classify each `<check>`:
- `code` — caused by the PR's code (compile error, test failure, lint) → fix in Step 8.
- `flaky` — transient (network timeout, race, known-flaky test) → re-run, don't fix code.
- `infra` — environment/runner (missing secret, service down, quota) → report; re-run only if user confirms.
- `required-status` — required check missing/pending (e.g. no recent review) → out of scope; report.
- Premise check — a `code` failure in code the PR doesn't touch, or one showing `<pr-intent>`'s assumed behavior no longer holds → verify against `<base-branch>`; pre-existing or drifted → `<not-fixing>` with reason. Don't widen scope silently.

Record `<classification>`; `flaky`/`infra`/`required-status` → `<not-fixing>` with reason. All failures non-`code` → summary + stop (suggest re-runs where applicable).

### 7. Plan Fix
Fast path: every `code` failure straightforward, no cross-check dependencies → `<implementation-plan>` = `trivial` — on this path partition `code` failures into `<fix-groups>` (shared file lands in one group) and mark `parallel-eligible` per the criteria below before skipping to Step 8 when dispatch is viable: three or more groups or the user's explicit request. Otherwise ordered fix list (compile/build → test → lint; then shared file, then check); sequential thinking merges failures sharing one root cause → `<dependencies>`; partition `code` failures into `<fix-groups>` (shared root cause or shared file lands in one group) and mark a group `parallel-eligible` when its files are disjoint from the other groups, it has no cross-group dependencies, and it runs no commands writing shared mutable state such as lockfiles or build caches.

### 8. Implement Fix
On `<active-branch>` — focused, minimal, existing patterns. `parallel-eligible` `<fix-groups>` may be implemented in parallel — dispatch one subagent per group, each editing only its assigned files — only with three or more groups or the user's explicit request to delegate, and only when a suitable subagent profile exists; shared-cause or doubtful-independence failures stay serial. Collect each group's changed files into `<fix-files>`, verifying they stayed within its boundaries; a failed group's changes are discarded or reset before validating or pushing — never mix successful and failed group edits; its checks are retried serially in this session, and checks still failing are recorded unresolved in `<validation-results>` and surfaced in the approval summary — never counted as fixed. Local validation, commit, and push stay serial. Root cause, not symptom: never disable a failing test — fix what it asserts unless the assertion is wrong (then fix it with justification) → `<changes-count>` and `<fix-files>`.

### 9. Validate Locally
Reproduce each fixed `<check>`'s `<failing-command>` locally. Set `<validation-passing>` to `no` if an executed check fails, `unverified` if any check cannot run or a failed group left a check unresolved, and `yes` only if all pass — unresolved checks stay listed in `<validation-results>` and the approval summary, never dropped. Record the details in `<validation-results>`.

### 10. Approval Gate (mandatory in `review` mode)
Same gate as `pr-fix` Step 9 — `review`/`auto` semantics, `Go Ahead`/`Needs Review` loop (refine → re-validate → re-present until `approved`; refinement that changes which files a fix touches re-derives `<fix-groups>` first). Present: failing checks fixed, file count, local validation, `<not-fixing>` items (flaky/infra with suggested re-runs). `<changes-count>` = 0 → surface no-change state.

### 11. Commit and Push (after `approved` or `auto`)
Inspect the worktree and index again, including changes within `<fix-files>`. Unexpected changes → STOP and preserve them. Stage and commit only `<fix-files>` with explicit pathspecs (message convention per `commit-and-push`), then `git push origin <active-branch>`. Never use a generic message like "fix ci". Do not poll CI or loop back — session ends after push.

## Ticket Brief

When a step calls for a ticket brief, reuse session results first — a prior brief covering the same references and scope, or the referenced tickets' content already loaded at the depth the brief requires (comments and attachments included; a lighter read does not qualify) → derive the scoped answers without delegating. Otherwise delegate to one isolated, read-only subagent: pass only the ticket references, the scope, `<project-root>`, and these rules — the main session receives only the returned brief. Worker: read `<project-root>/docs/issue-trackers.md` and use its read operations; load each in-scope ticket's body, checklists, full comment thread, and accessible attachments (inspect contents, not filenames); follow only links bearing on the scope; dedupe by tracker and ID. Return a compact cited brief — the scoped answers, confirmed decisions distinguished from proposals and superseded ones, blockers and evidence gaps; cite comment permalinks or attachment name plus location; omit raw threads and attachment contents; mark the brief partial when missing evidence could change it. Never post or modify tracker state. Worker or spec unavailable → report the gap without fetching inline.

## Output
Summary of fixed checks, local validation, pushed state — one line per fix at the gate, e.g. "Fixed N failing checks across M files" + validation + "pushed to `<active-branch>`". Never claim CI green — unverified.
