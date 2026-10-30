---
name: pr-fix-ci
description: "Activate when a pull request has failing CI checks that require code fixes."
---

## Required `<project-root>/docs` reads

Read via shell before fixing failing CI checks — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/git-hosts.md`

Fix failing CI checks on a PR: read logs, fix code, push, verify. Tools: git host (`<project-root>/docs/git-hosts.md`). Check source: `gh pr checks` / `gh pr view --json statusCheckRollup`.

Modes: `review` (default) or `auto` — `auto` skips the Step 10 approval gate. `<pr-ref>` is the only authoritative source for the target PR — never infer from the current branch.

### 1. Interpret Arguments
`auto` → `<execution-mode>` = `auto`; PR number/URL → `<pr-ref>`; extra guidance → `<additional-context>`; otherwise → `review`.

### 2. Load PR Context
Verify git host auth (`<project-root>/docs/git-hosts.md`). `git rev-parse --show-toplevel` → `<repo-path>` (use as `cwd`). View PR via git host view PR (includes checks) or fallback:
```bash
gh pr view <pr-number> --json number,title,headRefName,baseRefName,url,state,statusCheckRollup
```
Extract `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`. `git branch --show-current` → `<current-branch>`.

### 3. Align Local Branch
`<pr-branch>` unavailable → STOP. Check out via git host checkout PR or `gh pr checkout <pr-number>` → `<active-branch>`; fails → STOP. No code edits until `<active-branch>` == `<pr-branch>`.

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

Record `<classification>`; `flaky`/`infra`/`required-status` → `<not-fixing>` with reason. All failures non-`code` → summary + stop (suggest re-runs where applicable).

### 7. Plan Fix
Fast path: every `code` failure straightforward, no cross-check dependencies → `<implementation-plan>` = `trivial`, skip to Step 8. Otherwise ordered fix list (compile/build → test → lint; then shared file, then check); merge failures sharing one root cause → `<dependencies>`.

### 8. Implement Fix
On `<active-branch>` — focused, minimal, existing patterns. Root cause, not symptom: never disable a failing test — fix what it asserts unless the assertion is wrong (then fix it with justification) → `<changes-count>`.

### 9. Validate Locally
Reproduce each fixed `<check>`'s `<failing-command>` locally (lint/build/test on changed files); confirm green. A check that can't run locally → state so, rely on Step 12 → `<validation-results>`, `<validation-passing>` (`yes`/`no`).

### 10. Approval Gate (mandatory in `review` mode)
Never skip — only `decision: approved` counts. `<validation-passing>` = `no` → STOP. `auto` → skip to Step 11.

Present gate: failing checks fixed, changed file count, local validation, `<not-fixing>` items (flaky/infra with suggested re-runs). Ask (header `Approval`): "Ready to commit and push?" `Go Ahead` → `approved`; `Needs Review` → `concerns` (requires feedback).

- `approved` → Step 11. `concerns` → `<review-feedback>` → refine (8) → re-validate (9) → re-present. Repeat until `approved`.
- `<changes-count>` = 0 (all non-`code`) → surface no-change state so user picks `Go Ahead` (no-op) or `Needs Review`.

### 11. Commit and Push (after `approved` or `auto`)
`git add -A`, `git commit -m "<message>"` (lowercase, `<prefix>: <summary>`, under 50 chars), `git push origin <active-branch>`. Never generic messages like "fix ci". Do not poll CI or loop back — session ends after push.

## Output
Summary of fixed checks, local validation, pushed state — one line per fix at the gate, e.g. "Fixed N failing checks across M files" + validation + "pushed to `<active-branch>`". Never claim CI green — unverified.
