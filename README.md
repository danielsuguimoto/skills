# Skills

Agent skills repository for use with any agentic tool that supports skill manifests — Devin CLI, Claude Code, Cursor, OpenCode, Codex, Gemini CLI, and others.

Each subdirectory under `skills/` is a self-contained skill with a `SKILL.md` manifest describing its name, description, and instructions.

## Included skills

| Skill | Description |
|-------|-------------|
| browser-automation | Browser automation — QA, element interaction, screencapture, network capture, cross-account/app/memory/history access. |
| caveman | Compression communication mode. Drops filler, hedging, and narration while keeping full sentences and technical accuracy. |
| code-work | Code reading and editing conventions — scoped reads, minimal edits, no comments. |
| commit-and-push | Commit and push current changes. |
| context-gathering | MANDATORY pre-flight before any codebase exploration, reading, editing, or answering about code. |
| database | Database inspection tools for schema, queries, and operations. |
| debug | Troubleshoot and fix a user-reported bug — grill to clarify, investigate root cause, plan, and execute the fix. |
| adjust | Apply user-requested adjustments to existing code — fixes, improvements, refactors per notes. No shipping. |
| knowledge-graph | Turn any input (code, docs, papers, images, videos) into a persistent knowledge graph with community detection and query/path/explain tools. |
| grill-with-codebase | Grill the user about a task or intent using the codebase as ground truth until decisions form a concrete plan. |
| implement | Develop from a ticket — load details, gather context, implement on master. |
| container-cli | Container CLI usage reference — start/stop/restart, exec, logs, run scripts, share tunnels. |
| memory-usage | Read or write persistent project memory before, during, or after non-trivial work. |
| new-branch | Create and switch to a categorized branch summarizing current uncommitted or ahead-of-base work. |
| pr-create | Create a pull request for the current branch. |
| pr-fix | Address PR feedback by making fixes and responding directly to review threads. |
| pr-fix-ci | Fix failing CI checks on a PR — read logs, fix the code, push, verify green. |
| session-to-memory | Extract retention-worthy knowledge from a session and persist it to the active memory provider. |
| setup-skills | Guide users through initial configuration — creates /docs folder and generates all spec files needed by other skills. |
| ship-changes | Ship current work end-to-end — branch, commit, push, open PR. |
| spawn | Invoke a target skill inside a subagent so its content stays isolated to the subagent's context. |
| subagents | Subagent spawning rules for parallel work, isolated tasks, and context offloading. |
| to-specs | Turn a request or ticket into a spec, then sync to the ticket system. Read-only. |
| triage | Triage a ticket — load context, gather evidence, post findings. Read-only. |
| using-code-navigation | Use code navigation tools for symbol-level code work instead of grep/read. |
| using-terminal | MANDATORY pre-flight before sending any command to the terminal/shell. |

## Tool Specifications (`/docs`)

Skills are tool-agnostic. They reference tool specifications in a `/docs` folder that lives in the **project root** of each consuming project — not in this skills repo.

Each project using these skills must create a `/docs` folder with markdown files documenting the interface and expected behavior of the tools available in that project. Skills reference these specs by path (e.g., `/docs/issue-trackers.md`), allowing agents to accomplish workflows using any tool implementation that conforms to the documented interface.

### Expected spec files

| File | Covers |
|------|--------|
| `/docs/issue-trackers.md` | Issue/ticket tracking tools — load, sync, list tickets |
| `/docs/bug-trackers.md` | Bug tracking and error monitoring tools — query errors, stack traces |
| `/docs/mcp-servers.md` | MCP server interaction — list tools, call tools, discovery protocol |
| `/docs/git-hosts.md` | Git hosting platforms — PRs, reviews, issues, branches, commits |
| `/docs/database-tools.md` | Database inspection — schema, queries, REPL operations |
| `/docs/code-navigation.md` | LSP-backed code navigation — find symbols, trace references, refactor |
| `/docs/browser-automation.md` | Browser automation — snapshots, element interaction, downloads |
| `/docs/knowledge-graphs.md` | Knowledge graph tools — build, query, path, explain |
| `/docs/container-clis.md` | Container CLI wrappers — lifecycle, exec, logs, scripts |
| `/docs/memory-providers.md` | Persistent memory — list, read, write, edit, delete memories |
| `/docs/terminal-wrappers.md` | Terminal token compression wrappers — command prefixing, output filtering |
| `/docs/doc-lookup.md` | Documentation lookup — framework/library/SDK docs, search fallbacks |

Each spec file must define: the operations available, their input/output schemas, and expected behavior rules. Agents read these files to learn how to interact with the project's specific tool implementations.

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
