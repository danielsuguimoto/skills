---
name: setup-skills
description: >
  Guide users through initial configuration to enable all danielsuguimoto/skills
  skills. Creates the /docs folder and generates spec files for each tool category.
  Invoke when the user says 'setup skills', 'configure skills', 'initialize skills',
  or when /docs spec files are missing or incomplete.
---

# Setup Skills

Generate the `/docs` folder and all 12 spec files in the project root so every danielsuguimoto/skills skill can function. Skills reference these specs by path (e.g., `/docs/git-hosts.md`); without them, skills cannot locate the tools they need.

## Pre-flight

1. **Detect project root**: `git rev-parse --show-toplevel` or fall back to `cwd`. Store as `<project-root>`.
2. **Check existing state**: List `<project-root>/docs/` if it exists. Check which of the 12 expected spec files already exist.
3. **Decision gate**:
   - All 12 exist → ask user whether to reconfigure or abort. If abort, stop.
   - Some exist → ask user whether to regenerate all or only missing ones.
   - None exist → proceed directly to interview.
4. **Discover MCP servers**: Use native MCP server discovery if available. Store as `<mcp-inventory>`. Offer detected servers as options during the interview. Skip if unsupported.

## Interview

Ask the user about each tool category using the agent's native user-prompting mechanism. Three batches of four questions. Each question offers common options plus a "None / not used" option. Include the detected MCP server as an option when relevant. Present one batch at a time. Wait for all answers before proceeding to the next batch. Record every answer as `<tool-selections>`.

### Batch 1 — Tracking, Git, Database

| Question | Header | Options |
|----------|--------|---------|
| What issue/ticket tracker do you use? | Issue tracker | Trello, Linear, Jira, None |
| What bug/error monitoring tool do you use? | Bug tracker | Sentry, Bugsnag, Rollbar, None |
| What git hosting platform do you use? | Git host | GitHub, GitLab, Bitbucket, None |
| What database inspection tools do you use? | Database | Laravel Boost MCP, Direct SQL client, None |

### Batch 2 — Code, Browser, Containers

| Question | Header | Options |
|----------|--------|---------|
| What code navigation/LSP tool do you use? | Code navigation | Serena, Sourcegraph, None |
| What browser automation tool do you use? | Browser | Playwright, Aside, None |
| What knowledge graph tool do you use? | Knowledge graph | Graphify, None |
| What container CLI do you use? | Container CLI | Kool, Docker Compose, None |

### Batch 3 — Memory, Terminal, Docs, MCP

| Question | Header | Options |
|----------|--------|---------|
| What persistent memory provider do you use? | Memory | Serena memory, Mem0, None |
| What terminal token compression wrapper do you use? | Terminal | RTK, None |
| What documentation lookup tools do you use? | Doc lookup | Context7, Tavily, None |
| Which MCP servers should be documented? | MCP servers | Auto-detect from `<mcp-inventory>`, None |

For the MCP servers question: if `<mcp-inventory>` is non-empty, offer "Use detected servers" first. If empty, offer "None" and "Other".

## Generate

Create `<project-root>/docs/` if missing. For each of the 12 spec files, generate the file. If the user selected "None", write a stub: `**Tool**: Not configured` with a note that skills referencing this spec will use fallbacks or skip.

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

The 12 spec files and what they cover (see danielsuguimoto/skills README for the canonical list):

| File | Covers |
|------|--------|
| `/docs/issue-trackers.md` | Load, sync, list tickets |
| `/docs/bug-trackers.md` | Query errors, stack traces |
| `/docs/mcp-servers.md` | List tools, call tools, discovery protocol |
| `/docs/git-hosts.md` | PRs, reviews, issues, branches, commits |
| `/docs/database-tools.md` | Schema, queries, REPL operations |
| `/docs/code-navigation.md` | Find symbols, trace references, refactor |
| `/docs/browser-automation.md` | Snapshots, element interaction, downloads |
| `/docs/knowledge-graphs.md` | Build, query, path, explain |
| `/docs/container-clis.md` | Lifecycle, exec, logs, scripts |
| `/docs/memory-providers.md` | List, read, write, edit, delete memories |
| `/docs/terminal-wrappers.md` | Command prefixing, output filtering |
| `/docs/doc-lookup.md` | Framework/library/SDK docs, search fallbacks |

For `/docs/mcp-servers.md`: populate a server inventory section from `<mcp-inventory>` if the user chose to document detected servers.

## Verify and Output

Confirm all 12 spec files exist. Report summary table: spec file, tool, configured or not. List files with `<!-- Fill in: ... -->` markers needing manual completion. Do not commit or push.
