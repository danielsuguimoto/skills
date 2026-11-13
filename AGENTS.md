# Agent Guidelines

Skills repository: `skills/<name>/SKILL.md` manifests plus the `README.md` skill index. Consumers are AI agents — a skill's prose is the product. `docs/`, `.devin/`, `.serena/` are gitignored local state. No build or test suite; verify by reading.

## Conventions

- Write agent-facing instructions per `writing-for-agents` and `skills/caveman/SKILL.md`: compressed, grammatical, and clear.
- A skill's frontmatter `description` names every distinct invocation case without redundant synonyms or workflow details.
- Keep bodies focused: prune duplication, stale content, and no-ops; disclose branch-specific material without hiding required steps or safety rules.
- Minimize execution reads, tool calls, reasoning, user questions, and chat output; preserve evidence, correctness, and required approvals.
- `<angle-bracket>` placeholders for variables; `path:line` for file references.
- Tool names and CLIs live in `docs/` specs — skill bodies name spec files, not tools. The canonical spec list is the Generate table in `skills/setup-skills/SKILL.md`.
- Single source of truth — a rule or convention lives in one file; restated rules drift.
- Added, renamed, or removed skills update the `README.md` table.

## Git

- Commits: `<prefix>: <message>` — lowercase, under 50 chars. Prefixes: feat, fix, chore, docs, refactor, test, style.

## Pointers

- PR review checks: `REVIEW.md`.
- Auto-approval contract: `APPROVAL_POLICY.md`.
