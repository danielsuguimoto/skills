# Approval Policy

Instructions for Devin Review's Autoapprover on this repository. Devin Review reads this file from the target branch; a PR that modifies it always requires human review.

## Approve without a human

Approve when review completed and every condition holds:

- No open findings — bugs, security findings, or investigate flags the reviewer posted. An unavailable findings signal counts as no findings when review completed and posted none; a review that never ran is not a clean review.
- The diff stays inside the PR's stated scope — no unrelated files touched.
- Every check in `REVIEW.md` passes: triggers intact, spec pointers valid, single source of meaning, `README.md` index synced, completion criteria checkable.
- Verify the PR's checks yourself: a failing check blocks approval; absent, pending, or unavailable check status does not.

## Findings, not blanket escalations

Capability wording is review evidence, not an automatic veto: every skill edit changes what agents may do, so widened authority alone is not grounds for escalation. Flag concrete dangers as findings — the `REVIEW.md` prompt-safety cases, plus skill wording that adds unneeded network or credential access. A flagged danger is an open finding and blocks approval; unflagged wording approves like any other diff.
