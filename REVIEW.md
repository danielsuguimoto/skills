# Review Guidelines

Instructions for Devin Review on pull requests in this repository. Repository shape and authoring conventions live in `AGENTS.md` — verify the diff against them.

## Strategy

1. **Scope** — every changed file serves the PR's stated intent. Flag unrelated edits; repo rule is exact scope.
2. **Triggers** — verify the `AGENTS.md` trigger convention on changed skills; a weakened trigger means the skill silently stops firing.
3. **Spec pointers** — verify the `AGENTS.md` spec-pointer convention on `docs/` references (the Generate table is the contract for consuming projects).
4. **Single source** — flag violations of the `AGENTS.md` single-source rule.
5. **Index sync** — flag violations of the `AGENTS.md` index-sync rule.
6. **Completion criteria** — flag vague step bounds ("make sure it's correct"); steps need checkable done-conditions, flat reference needs an exhaustiveness bar ("every X").

## Critical areas

- **Mandatory-skill wording** (`context-gathering`, `using-terminal`, `subagents`, `code-work`) — small edits alter every consuming session.
- **Prompt safety** — skill bodies are injected into agent prompts verbatim: flag instructions that read secrets into output, exfiltrate data, or run destructive commands without confirmation.
- **Spec contract** — changes to `skills/setup-skills/SKILL.md`'s Generate table ripple to every consuming project's docs.

## Ignore

- `.serena/`, `.devin/`, `docs/` — local tool state; gitignored.
- Whitespace-only churn.

## Approval

No findings and the diff matches its stated intent → approve rather than leave pending. The auto-approval contract lives in `APPROVAL_POLICY.md`.
