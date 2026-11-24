---
name: setup-skills
description: "Guide initial configuration for this skills system — creates <project-root>/docs spec files per tool category. Invoke on 'setup skills', 'configure skills', 'initialize skills', or when spec files are missing or incomplete."
---

# Setup Skills

Generate `<project-root>/docs/` and every spec file in the Generate table — skills reference specs by path to locate tools.

## Pre-flight

1. **Detect project root**: `git rev-parse --show-toplevel` or fall back to `cwd` → `<project-root>`.
2. **Check existing state**: list `<project-root>/docs/`; check which Generate-table specs exist.
3. **Decision gate**: all exist → ask reconfigure or abort (abort → stop) · some → ask regenerate all or only missing · none → interview directly.
4. **Discover MCP servers**: native MCP discovery if available → `<mcp-inventory>`; offer detected servers as interview options. Skip if unsupported.
5. **Detect external agent CLIs**: probe PATH for agent harness binaries (e.g. `claude`, `gemini`, `codex`, `devin`) → `<agent-cli-inventory>`; offer detected CLIs as interview options. If PATH enumeration is unsupported, skip detection; an empty inventory still flows to the interview.
6. **Enumerate LLM models**: list models via the harness's model-listing mechanism (model picker, config file, provider API) → `<model-inventory>`; skip if unsupported.

## Interview

Ask about each tool category via the native user-prompting mechanism — three sequential batches. Each question offers common options plus "None / not used"; include detected MCP servers when relevant. Record answers as `<tool-selections>`.

### Batch 1 — Tracking, Observability, Git, Database

| Question | Header | Options |
|----------|--------|---------|
| What issue/ticket tracker do you use? | Issue tracker | Trello, Linear, Jira, None |
| What bug/error monitoring tool do you use? | Bug tracker | Sentry, Bugsnag, Rollbar, None |
| What monitoring/incident platform do you use? | Observability | Datadog, Flare, Grafana, None |
| Where do production deploy/job logs live? | Deploy logs | Same as observability, Git host CI, None |
| What git hosting platform do you use? | Git host | GitHub, GitLab, Bitbucket, None |
| What database inspection tools do you use? | Database | Laravel Boost MCP, Direct SQL client, None |

### Batch 2 — Code, Browser, Containers

| Question | Header | Options |
|----------|--------|---------|
| What code navigation/LSP tool do you use? | Code navigation | Sourcegraph, None |
| What browser automation tool do you use? | Browser | Playwright, Aside, None |
| What knowledge graph tool do you use? | Knowledge graph | Graphify, None |
| What container CLI do you use? | Container CLI | Kool, Docker Compose, None |

### Batch 3 — Memory, Terminal, Workspaces, Docs, MCP, External Agents, Models

| Question | Header | Options |
|----------|--------|---------|
| What persistent memory provider do you use? | Memory | Mem0, None |
| What terminal token compression wrapper do you use? | Terminal | RTK, None |
| What isolated workspace tool do you use? | Workspaces | git worktree, None |
| What documentation lookup tools do you use? | Doc lookup | Context7, Tavily, None |
| Which MCP servers should be documented? | MCP servers | Auto-detect from `<mcp-inventory>`, None, Other |
| Which external agent CLIs should be documented? | External agents | Auto-detect from `<agent-cli-inventory>`, None, Other |
| Which LLM models should be documented? | Models | Auto-detect from `<model-inventory>`, None, Other |

MCP servers question: `<mcp-inventory>` non-empty → "Use detected servers" first; empty → "None" and "Other".

External agents question: `<agent-cli-inventory>` non-empty → "Use detected CLIs" first; empty → "None" and "Other".

Models question: `<model-inventory>` non-empty → "Use detected models" first; empty → "None" and "Other".

## Generate

Create `<project-root>/docs/` if missing. Generate each spec file below, filling every Operations row and Tool Details line derivable from the selected tool; keep `<!-- Fill in: ... -->` markers only where the interface can't be derived. "None" selected → stub: `**Tool**: Not configured` + note that referencing skills use fallbacks or skip.

Each spec documents the tool's interface only — no agent behavior rules:

```markdown
# <Category>

**Tool**: <tool-name or "Not configured">

## Operations

| Operation | Purpose | Inputs | Outputs |
|-----------|---------|--------|---------|
<!-- Fill in: one row per operation the tool exposes -->

## Tool Details

<!-- Fill in: MCP server name + tool names, or CLI commands, or API endpoints -->
```

Spec files:

| File | Covers |
|------|--------|
| `<project-root>/docs/issue-trackers.md` | Load, sync, list tickets |
| `<project-root>/docs/bug-trackers.md` | Query errors, stack traces |
| `<project-root>/docs/observability-tools.md` | Query alerts, incidents, uptime, deploy and job logs |
| `<project-root>/docs/mcp-servers.md` | List tools, call tools, discovery protocol |
| `<project-root>/docs/git-hosts.md` | PRs, reviews, issues, branches, commits, sync |
| `<project-root>/docs/database-tools.md` | Schema, queries, REPL operations |
| `<project-root>/docs/code-navigation.md` | Find symbols, trace references, refactor |
| `<project-root>/docs/browser-automation.md` | Snapshots, element interaction, downloads |
| `<project-root>/docs/knowledge-graphs.md` | Build, query, path, explain |
| `<project-root>/docs/container-clis.md` | Lifecycle, exec, logs, scripts |
| `<project-root>/docs/workspace-isolation.md` | Detect, create, list, remove isolated workspaces |
| `<project-root>/docs/memory-providers.md` | List, read, write, edit, delete memories |
| `<project-root>/docs/terminal-wrappers.md` | Command prefixing, output filtering |
| `<project-root>/docs/terminal-tools.md` | Non-standard terminal tools replacing usual defaults |
| `<project-root>/docs/doc-lookup.md` | Framework/library/SDK docs, search fallbacks |
| `<project-root>/docs/external-agents.md` | Other agent harness CLIs — headless invocation, output capture, auth |
| `<project-root>/docs/models.md` | LLM models — inventory, tier/cost, selection mechanism (picker, config, profile pins) |

`mcp-servers.md`: populate a server inventory section from `<mcp-inventory>` if the user chose to document detected servers.

`external-agents.md`: populate a harness inventory from `<agent-cli-inventory>`; per harness document the headless invocation, output capture, and auth requirements.

`models.md`: populate a model inventory from `<model-inventory>`; per model document tier (fast/cheap, mid, strong), cost or rate notes where known, and how the harness selects it for subagents (profile `model:` pin, default-subagent-model config, inheritance).

`observability-tools.md`: when the deploy/job-log source differs from the monitoring tool, document that source's lookup operations and details alongside the monitoring tool's.

## Verify and Output

Confirm every spec file exists. Report a summary table: spec file, tool, configured or not. List files still carrying `<!-- Fill in: ... -->` markers as manual-completion gaps. Do not commit or push.
