---
name: pr-watch
description: "Watch a PR for review feedback and failing CI checks until review and checks are clean."
---

## Required `<project-root>/docs` reads

Shell-read before watching a PR (reuse session reads): `<project-root>/docs/git-hosts.md`. If the spec is missing, report the gap and use a documented fallback.

## 1. Select the PR

Resolve `<pr-ref>` from the explicit PR number/URL, else the unambiguous PR targeted by the session's current work. Missing or ambiguous → ask; never select an unrelated PR. Record the repository, PR number, URL, and current head. Pass `auto` to fix skills only when the user explicitly supplied `auto`; otherwise use their default approval gates. Verify git host access and that `pr-fix` and `pr-fix-ci` are available; if either is missing, ask the user to install it and stop.

## 2. Observe

On each pass, reload the PR state and head, all unresolved review threads, reviews, flat comments, review requests, and checks via `git-hosts.md`; paginate where needed. Identify expected reviews from the PR's review requests, required review policy, and review automation visible for the PR. Identify expected checks from host policy and applicable CI configuration, even when some checks have already reported. Associate results with the head they examined; missing expected checks are not started. A head change invalidates previous clean review and check results. Recheck the head after the reads; if it changed, discard the snapshot and reload before routing.

Collect feedback from threads, nonempty review bodies, and distinct flat comments, including bot findings; exclude own-agent items and CI, coverage, or deployment notices. Deduplicate restatements. Track item identity, updates, and fix-skill outcomes across passes so already assessed flat feedback is not dispatched again unless changed. Track pending review and check runs separately from failures.

## 3. Route

Before dispatch, a closed or merged PR → stop; report a clean verdict only if the clean conditions were already verified before closure.

- New or updated feedback, or an unresolved thread needing attention → invoke `pr-fix` for `<pr-ref>`. Preserve its scope, validation, publication, reply, and resolution gates.
- Failing checks → invoke `pr-fix-ci` for `<pr-ref>`. Preserve its triage, validation, and publication gates.
- When both exist, handle feedback first, then reload before routing checks. After either skill returns, reload the PR before deciding what remains; a push starts a new head's review and check cycle.

Invoke each skill serially. If a required skill is unavailable, an approval is declined, a fix is blocked or deferred, or the same unchanged feedback or failure remains after a completed attempt, stop and report the item and reason rather than repeatedly invoking it. Never treat an unanswered reviewer decision, failed reply or resolution, or unverified fix as clean.

## 4. Wait or Finish

Finish clean only on a fresh, consistent snapshot of the current head: every expected review has completed on that head, no outstanding changes request, unresolved thread, or unaddressed feedback remains, and every expected check has completed successfully. Verify that no checks are configured before treating an empty check list as clean. A review that never ran is not a clean result. Unavailable review/check evidence or an unknown expected review or check set → stop and report the evidence gap.

If evidence shows a required check's producer cannot run for this PR, stop and report the blocker. An outstanding changes request with no actionable feedback → wait for a trackable reviewer decision; if it cannot be tracked, stop and report the blocker. A retained older-head approval with an expected review still missing current-head evidence and no trackable run or request for that review → stop and report that a fresh review is needed. Otherwise, while expected reviews or checks have not yet started or are pending on the current head, wait 60 seconds between refreshes, respecting host rate limits, then repeat Steps 2–4. Report the PR, head, review and check results, and any blockers.
