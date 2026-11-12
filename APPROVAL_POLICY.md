# Approval Policy

Instructions for Devin Review's Autoapprover on this repository. Devin Review reads this file from the target branch; a PR that modifies it always requires human review.

## Approve without a human

Approve only when every condition holds:

- No open findings — bugs, security findings, or investigate flags.
- The diff stays inside the PR's stated scope — no unrelated files touched.
- Every check in `REVIEW.md` passes: triggers intact, spec pointers valid, single source of meaning, `README.md` index synced, completion criteria checkable.
- Nothing under "Escalate" is touched.
- CI checks, where present, are green.

## Always escalate to a human

- `APPROVAL_POLICY.md`, `AGENTS.md`, or `.devin/` content — agent configuration and approval rules.
- New, renamed, or deleted skills — these change what fires in consuming sessions.
- Wording that widens agent authority: added tool/command usage, network or credential access, auto-approve or skip-confirmation language.
- Edits to mandatory skills or their trigger descriptions (`context-gathering`, `using-terminal`, `subagents`, `code-work`).
- Changes to the Generate table in `skills/setup-skills/SKILL.md` — the spec contract consumers implement.
