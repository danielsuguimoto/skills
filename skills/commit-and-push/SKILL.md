---
name: commit-and-push
description: "Commit and push current changes."
---

## Required `<project-root>/docs` reads

Shell-read before committing or pushing; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`

## Operations

| Operation | Tool (`git-hosts.md`) | CLI fallback |
|-----------|--------------|--------------|
| Load change state | git host change scan | change-scan fallback per spec |
| Stage and commit | git host commit (with `message` and file list) | `git add -- <included>` + `git commit -m "<message>" -- <included>` |
| Push to remote | git host push | `git push` (retry `git push -u origin <branch>` if no upstream) |

### 1. Interpret Arguments

- Wording influencing commit message → `<additional-context>`
- Files to leave out (forwarded by an orchestrator or named by the user) → `<excluded>`
- Otherwise undefined

### 2. Load Changes

If a pre-scan was already forwarded:
- Use `<uncommitted>` (full set) and `<ahead-commits>` directly.
- Compute `<included>` = changed files in `<uncommitted>` that are not in `<excluded>`.
- If `<change-summary>` was not forwarded, build it from `<included>` and `<additional-context>`.

Otherwise:

- **2a.** Load uncommitted changes (status + file list + diff stat) via the operations table above. If file paths are missing, collect the full changed-file list (untracked included) via the change-scan fallback (`git-hosts.md`). Store `<uncommitted>`.
- **2b.** If `<uncommitted>` is empty, load commits ahead using `@{u}..HEAD`. Store `<ahead-commits>`.
- **2c.** Compute `<included>` = changed files in `<uncommitted>` that are not in `<excluded>`.
- **2d.** Analyze scope from `<included>` (primary) or `<ahead-commits>` (push only, no commit):
  - Group into logical themes. Summarize "what" and "why", not "how".
  - Store as `<change-summary>`, including `<additional-context>`.

### 3. Check Blockers

- If `<included>` and `<ahead-commits>` are both empty, STOP: "Nothing to commit or push"

### 4. Create Commit

Skip if `<included>` is empty (push ahead commits only).

Commit message: `<prefix>: <message>` — lowercase, under 50 chars, no body. Prefixes: feat, fix, chore, docs, refactor, test, style.

Commit:
1. Stage `<included>` explicitly (file paths from Step 2). A staged `<excluded>` path must not enter the commit — unstage it or keep the commit pathspec-limited to `<included>`; a committed excluded file reaches the remote on push.
2. Generate message from `<change-summary>` and `<additional-context>`.
3. Commit via git host commit or `git commit -m`; store `<hash>`.

### 5. Push to Remote

Record the push destination's remote tip before pushing → `<remote-tip>` (`@{u}`, else `origin/<branch>`). Push via git host push or `git push`; no upstream → `git push -u origin <branch>`. Rejected non-fast-forward (remote moved) → fetch the pushed-to remote ref — not `@{u}` when it names a different ref; a rejected first `push -u` → fetch `origin <branch>` — then `git rebase --onto <new-tip> <remote-tip>`: only local commits replay, a plain rebase could republish replaced upstream commits; linear range only (no merge commits): clean → retry the push once; conflict → `git rebase --abort`, STOP and report — never force-push. Any other failure → STOP, report the error. Success → output HEAD's `<hash>` + `<push-target>`.

