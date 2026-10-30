---
name: commit-and-push
description: "Commit and push current changes."
---

## Required `<project-root>/docs` reads

Read via shell before committing or pushing — may be gitignored. Missing → note the gap; never invent contents.

- `<project-root>/docs/git-hosts.md`

## Operations

| Operation | Tool (`<project-root>/docs/git-hosts.md`) | CLI fallback |
|-----------|--------------|--------------|
| Load change state | git host change scan | `git branch --show-current` + `git status --short` + `git diff --stat` + `git diff --cached --stat` + `git log @{u}..HEAD --oneline` |
| Stage and commit | git host commit (with `message`) | `git add -A` + `git commit -m "<message>"` |
| Push to remote | git host push | `git push` (retry `git push -u origin <branch>` if no upstream) |

### 1. Interpret Arguments

- Wording influencing commit message → `<additional-context>`
- Otherwise undefined

### 2. Load Changes

If a pre-scan was already forwarded, skip to step 3.

Otherwise:

- **2a.** Load uncommitted changes via the operations table above. Store `<uncommitted>`.
- **2b.** If `<uncommitted>` is empty, load commits ahead using `@{u}..HEAD`. Store `<ahead-commits>`.
- **2c.** Analyze scope:
  - Primary `<uncommitted>`; fallback `<ahead-commits>` (push only, no commit).
  - Group into logical themes. Summarize "what" and "why", not "how".
  - Store as `<change-summary>`, including `<additional-context>`.

### 3. Check Blockers

- If `<uncommitted>` and `<ahead-commits>` are both empty, STOP: "Nothing to commit or push"

### 4. Create Commit

Skip if `<uncommitted>` is empty (push ahead commits only).

Commit message: `<prefix>: <message>` — lowercase, under 50 chars, no body. Prefixes: feat, fix, chore, docs, refactor, test, style.

Commit:
1. Stage (`-A` or specific files).
2. Generate message from `<change-summary>` and `<additional-context>`.
3. Commit via git host commit or `git commit -m`; store `<hash>`.

### 5. Push to Remote

Push via git host push or `git push`. If no upstream, retry `git push -u origin <branch>`. Store `<push-target>`. If push fails, STOP and report the error. On success, output `<hash>` and `<push-target>`.

