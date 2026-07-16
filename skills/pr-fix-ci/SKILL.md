---
name: pr-fix-ci
description: "Activate when a pull request has failing CI checks that require code fixes."
---

Fix failing CI checks on a PR by reading check logs, fixing the code, pushing, and verifying green. Tools: git host tool (see `/docs/git-hosts.md` in the project root). Check source: `gh pr checks` / `gh pr view --json statusCheckRollup` (CLI fallback per `/docs/git-hosts.md`).

Modes: `review` (default; approval required before commit/push) or `auto` (commits/pushes without approval). `<pr-ref>` is the only authoritative source for the target PR — never assume current branch determines it.

> CRITICAL: in `review` mode, get explicit approval (Step 10) before committing or pushing. Never infer approval.

### 1. Interpret Arguments
`auto` → `<execution-mode>` = `auto`; PR number/URL → `<pr-ref>`; extra guidance → `<additional-context>`; otherwise → `review`.

### 2. Load PR Context
Verify git host auth (see `/docs/git-hosts.md`). Resolve project root via `git rev-parse --show-toplevel` → `<repo-path>` (use as `cwd`). View PR via git host view PR (includes checks) or CLI fallback:
```bash
gh pr view <pr-number> --json number,title,headRefName,baseRefName,url,state,statusCheckRollup
```
Extract `<pr-branch>`, `<base-branch>`, `<pr-number>`, `<pr-url>`. Run `git branch --show-current` → `<current-branch>`.

### 3. Align Local Branch
If `<pr-branch>` unavailable → STOP. Check out via git host checkout PR or `gh pr checkout <pr-number>`. Store as `<active-branch>`. Checkout fails → STOP. Don't modify code until `<active-branch>` == `<pr-branch>`.

### 4. Identify Failing Checks
From `statusCheckRollup` (or `gh pr checks <pr-number>`), collect checks where state is `FAILURE`, `ERROR`, `CANCELLED`, or `TIMED_OUT`. Store each as `<check>` with `<name>`, `<state>`, `<run-id>` (Actions runs only), `<url>`. Pending/in-progress checks → wait and re-poll once before deciding (transient). No failing checks → output "No failing checks" and stop.

### 5. Read Failure Logs
For each `<check>`: Actions run → `gh run view <run-id> --log-failed` (failed steps only); fall back to `gh run view <run-id> --log` if empty. Non-Actions check → no CLI logs; open `<url>` and read the rendered failure output via `webfetch` if reachable, else record `<url>` for the user. Extract `<error-lines>` (first error/stack/assertion), `<failing-file>`, `<failing-test>`, `<failing-command>`.

### 6. Triage Failures
Classify each `<check>`:
- `code` — failure caused by the PR's code (compile error, test failure, lint violation). Fix in Step 8.
- `flaky` — transient (network timeout, race, known-flaky test). Re-run, don't fix code.
- `infra` — environment/runner issue (missing secret, service down, quota). Report; don't fix code. Re-run only if user confirms.
- `required-status` — required status check missing or pending (e.g., no recent review). Out of scope; report.

Record `<classification>` per check. Move `flaky`/`infra`/`required-status` to `<not-fixing>` with reason. If every failure is non-`code` → output summary and stop (suggest re-runs where applicable).

### 7. Plan Fix
Fast path: every `code` failure is a straightforward fix with no cross-check dependencies → `<implementation-plan>` = `trivial`, skip to Step 8. Otherwise: build ordered fix list (compile/build → test → lint; then shared file, then check). Merge failures sharing one root cause into one fix. Map dependencies → `<dependencies>`.

### 8. Implement Fix
Fix on `<active-branch>`, focused minimal changes, follow existing patterns. Address root cause, not the symptom (don't disable a failing test; fix what it asserts unless the assertion is wrong — then fix the assertion with justification). Store `<changes-count>`.

### 9. Validate Locally
Reproduce each fixed `<check>`'s `<failing-command>` locally (lint/build/test on changed files). Confirm green. If a check can't run locally (e.g., platform-specific), state so and rely on Step 12. Store `<validation-results>`, `<validation-passing>` (`yes`/`no`).

### 10. Approval Gate (mandatory in `review` mode)
Never skip. Only `decision: approved` counts. If `<validation-passing>` = `no` → STOP. If `auto` mode → skip to Step 11.

Present gate: failing checks fixed, changed file count, local validation results, any `<not-fixing>` items (flaky/infra with suggested re-runs). Ask (header `Approval`): "Ready to commit and push?" Options: `Go Ahead` (→ `approved`) or `Needs Review` (requires feedback → `concerns`).

- `approved` → Step 11. `concerns` → store `<review-feedback>` → refine (8) → re-validate (9) → re-present gate. Repeat until `approved`.
- When `<changes-count>` = 0 (all non-`code`), surface no-change state so user picks `Go Ahead` (no-op) or `Needs Review`.

### 11. Commit and Push (after `approved` or in `auto` mode)
`git add -A`, `git commit -m "<message>"` (lowercase, `<prefix>: <summary>`, under 50 chars), `git push origin <active-branch>`. Never use generic messages like "fix ci". Store `<pushed>` (`yes`/`no`).

### 12. Verify CI Green
Push triggers CI. Poll `gh pr checks <pr-number>` (or `gh pr view --json statusCheckRollup`) until all previously-failing `<check>`s are `SUCCESS`/`NEUTRAL`, or a reasonable timeout. For `flaky` checks, re-run the failed run via `gh run rerun <run-id> --failed` before polling. If a previously-passing check now fails (new regression), loop back to Step 4 with the new failure. Store `<ci-final>` (`green`/`failing`).

## Output
Summary of fixed checks, local validation, and final CI state. At the approval gate: one line per fix, not full diffs. Example: "Fixed N failing checks across M files" + local validation + (after Step 12) final CI status.

## Red Flags
**Never:**
- Commit or push before the approval gate in `review` mode
- Disable a failing test to make CI green — fix the root cause
- Assume current branch is the PR branch — `<pr-ref>` is authoritative
- Skip local validation (Step 9) when the failing command can run locally
- Re-run a `code` failure instead of fixing it

**Always:**
- Triage before fixing — not every red check is a code problem
- Cite the failing file/test/command from the actual log
- Poll CI to confirm green after push
