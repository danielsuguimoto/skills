# Skills

Agent skills repository for use with any agentic tool that supports skill manifests — Devin CLI, Claude Code, Cursor, OpenCode, Codex, Gemini CLI, and others.

Each subdirectory under `skills/` is a self-contained skill with a `SKILL.md` manifest describing its name, description, and instructions.

## Included skills

| Skill | Description |
|-------|-------------|
| aside-browser | Browser automation — QA, element interaction, screencapture, network capture, cross-account/app/memory/history access. |
| caveman | Compression communication mode. Drops filler, hedging, and narration while keeping full sentences and technical accuracy. |
| code-work | Code reading and editing conventions — scoped reads, minimal edits, no comments. |
| commit-and-push | Commit and push current changes. |
| context-gathering | MANDATORY pre-flight before any codebase exploration, reading, editing, or answering about code. |
| database | Laravel Boost MCP database tools and Tinker REPL for database inspection, analysis, and operations. |
| feedback | Apply one round of local code-review feedback after implement. Code fixes only — no PR, commit, or push. |
| graphify | Turn any input (code, docs, papers, images, videos) into a persistent knowledge graph with god nodes, community detection, and query/path/explain tools. |
| grill-with-codebase | Grill the user about a task or intent using the codebase as ground truth until decisions form a concrete plan. |
| implement | Develop from a ticket — load details, gather context, implement on master. |
| kool-cli | kool (kool.dev) CLI usage reference — start/stop/restart, exec, logs, run scripts, share tunnels. |
| memory-usage | Read or write persistent project memory before, during, or after non-trivial work. |
| new-branch | Create and switch to a categorized branch summarizing current uncommitted or ahead-of-base work. |
| pr-create | Create a pull request for the current branch. |
| pr-fix | Address PR feedback by making fixes and responding directly to review threads. |
| session-to-memory | Extract retention-worthy knowledge from a session and persist it to the active memory provider. |
| ship-changes | Ship current work end-to-end — branch, commit, push, open PR. |
| subagents | Subagent spawning rules for parallel work, isolated tasks, and context offloading. |
| to-specs | Turn a request or ticket into a spec, then sync to the ticket system. Read-only. |
| triage | Triage a ticket — load context, gather evidence, post findings. Read-only. |
| using-serena | Use Serena MCP for symbol-level code work instead of grep/read. |
| using-terminal | MANDATORY pre-flight before sending any command to the terminal/shell. |

## Installation

### Via npx skills (recommended)

Install all skills to your agent of choice:

```bash
npx skills add danielsuguimoto/skills
```

Install specific skills:

```bash
npx skills add danielsuguimoto/skills --skill caveman --skill code-work
```

Install globally to a specific agent:

```bash
npx skills add danielsuguimoto/skills -g -a claude-code -y
```

List available skills without installing:

```bash
npx skills add danielsuguimoto/skills --list
```

### Manual

Clone the repository:

```bash
git clone git@github.com:danielsuguimoto/skills.git
```

Then symlink or copy individual skill folders from `skills/` into your agent's skills path. Common locations:

| Tool | Path |
|------|------|
| Devin CLI | `~/.agents/skills/` |
| Claude Code | `~/.claude/skills/` |
| Cursor | `~/.cursor/skills/` |
| OpenCode | `~/.config/opencode/skills/` |
| Codex | `~/.codex/skills/` |

## License

See individual skill folders for license information.
