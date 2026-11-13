# Approval Policy

Instructions for Devin Review's Autoapprover on this repository. Devin Review reads this file from the target branch; a PR that modifies it always requires human review.

## Approve without a human

Approve when review completed and every condition holds:

- No open findings — bugs, security findings, or investigate flags the reviewer posted. An unavailable findings signal counts as no findings when review completed and posted none; a review that never ran is not a clean review.
- The diff stays inside the PR's stated scope — no unrelated files touched.
- Every check in `REVIEW.md` passes: triggers intact, spec pointers valid, single source of meaning, `README.md` index synced, completion criteria checkable.
- Verify the PR's checks yourself: a failing check blocks approval; absent, pending, or unavailable check status does not.

## Findings, not blanket escalations

- Approval keys on findings posted on the current diff — the approver does not escalate on a risk no finding names. A skill's subject matter is not a finding: this repo ships prose, not executable code, so a skill governing shipping, secrets, or deletion does not raise the approval bar on its own.
- Capability wording is review evidence, not an automatic veto: every skill edit changes what agents may do, so widened authority alone is not grounds for escalation. Flag concrete dangers as findings — the `REVIEW.md` prompt-safety cases, plus skill wording that adds unneeded network or credential access. A flagged danger blocks approval; unflagged wording approves like any other diff.
- A resolved finding stops blocking; its theme does not persist as a standing veto. A re-review with no open findings approves.
