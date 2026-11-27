---
name: setup-skills
description: "Configure skill specs and project references on request or when files are missing or incomplete."
---

# Setup Skills

Configure the docs in the Generate table.

## Pre-flight

1. **Detect project root**: `git rev-parse --show-toplevel` or fall back to `cwd` → `<project-root>`.
2. **Check existing state**: list `<project-root>/docs/`; check which Generate-table files exist.
3. **Choose scope**: all files exist → ask whether to reconfigure or stop; some exist → ask whether to regenerate all or fill gaps only; none → generate all. Stop if declined. Record the target files.
4. **Discover on demand**: discover MCP servers once for the MCP spec or any unresolved category question that can use them; reuse current session discovery as `<mcp-inventory>`. For targeted external-agent or model specs, record PATH detection as `<agent-cli-inventory>` or native model listing as `<model-inventory>`. Skip unsupported discovery.

## Interview

Ask only unresolved questions for target files, in native batches. Reuse supplied answers or existing configuration when it settles the question. Offer common choices, relevant detected tools, and a none option. Record selections as `<tool-selections>`.

### Batch 1 — Tracking, Observability, Git, Database

| Question | Header | Options |
|----------|--------|---------|
| What issue/ticket tracker do you use? | Issue tracker | Trello, Linear, Jira, None |
| What bug/error monitoring tool do you use? | Bug tracker | Sentry, Bugsnag, Rollbar, None |
| What monitoring/incident platform do you use? | Observability | Datadog, Flare, Grafana, None |
| Where do production deploy/job logs live? | Deploy logs | Same as observability, Git host CI, None |
| What git hosting platform do you use? | Git host | GitHub, GitLab, Bitbucket, None |
| What database inspection tools do you use? | Database | Laravel Boost MCP, Direct SQL client, None |

### Batch 2 — Code, Browser, Containers, Testing

| Question | Header | Options |
|----------|--------|---------|
| What code navigation/LSP tool do you use? | Code navigation | Sourcegraph, None |
| What browser automation tool do you use? | Browser | Playwright, Aside, None |
| What knowledge graph tool do you use? | Knowledge graph | Graphify, None |
| What container CLI do you use? | Container CLI | Kool, Docker Compose, None |
| Does the project have testing requirements beyond standard local startup/browser execution? | Testing | Yes, No |

### Batch 3 — Memory, Terminal, Workspaces, Docs, MCP, External Agents, Models

| Question | Header | Options |
|----------|--------|---------|
| What persistent memory provider do you use? | Memory | Mem0, None |
| What terminal token compression wrapper do you use? | Terminal | RTK, None |
| What isolated workspace tool do you use? | Workspaces | git worktree, None |
| What documentation lookup tools do you use? | Doc lookup | Context7, Tavily, None |
| Which MCP servers should be documented? | MCP servers | Auto-detect from `<mcp-inventory>`, None |
| Which external agent CLIs should be documented? | External agents | Auto-detect from `<agent-cli-inventory>`, None, Other |
| Which LLM models should be documented? | Models | Auto-detect from `<model-inventory>`, None, Other |

MCP servers question: `<mcp-inventory>` non-empty → "Use detected servers" first; empty → "None" and "Other".

External agents question: `<agent-cli-inventory>` non-empty → "Use detected CLIs" first; empty → "None" and "Other".

Models question: `<model-inventory>` non-empty → "Use detected models" first; empty → "None" and "Other".

A Yes answer to Testing → batch focused follow-ups for unresolved testing sections: startup, test data and actors, browser and environment, external services, and supporting checks. Record `<testing-requirements>`. No or unanswered → generate a stub.

## Generate

Create `<project-root>/docs/` if missing. Generate the target files from the table; leave other files unchanged. For tool specs, fill every Operations row and Tool Details line derivable from the selected tool; keep `<!-- Fill in: ... -->` markers only where the interface can't be derived. "None" selected for a tool → stub: `**Tool**: Not configured` + note that referencing skills use fallbacks or skip. No or unanswered testing requirements → a stub recording no project-specific requirements.

Each tool spec documents the tool's interface only — no agent behavior rules. `testing.md` is the exception: it records project-specific testing environment facts and constraints using the structure in its note below. Tool-spec template:

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
| `<project-root>/docs/testing.md` | Project-specific test startup, data, actors, environment constraints, supporting checks |
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

`external-agents.md`: populate a harness inventory from `<agent-cli-inventory>` plus manually named "Other" entries; per harness document the headless invocation, output capture, and auth requirements.

`models.md`: populate a model inventory from `<model-inventory>` plus manually named "Other" entries; per model document tier (fast/cheap, mid, strong), cost or rate notes where known, and how the harness selects it for subagents (profile `model:` pin, default-subagent-model config, inheritance).

`observability-tools.md`: when the deploy/job-log source differs from the monitoring tool, document that source's lookup operations and details alongside the monitoring tool's.

`testing.md`: use sections `Startup`, `Test data and actors`, `Browser and environment`, `External services`, and `Supporting checks`. Populate them from `<testing-requirements>` and existing project testing docs. Record access requirements without secrets or credentials.

`issue-trackers.md`: include a `Scoped ticket brief` operation — inputs: ticket refs, scope, `<project-root>`, optional target PR; outputs: cited answers and decision statuses. Prescribe this procedure for it: dedupe by tracker and ID; read title and the body, checklists, or metadata needed for the scope. Read the latest comment page when scope includes behavior or constraints. Read older comments, attachments, and linked tickets only to close a scope gap or settle an unclear decision; this overrides general tracker traversal. For the target PR, return relevant decision references without fetching its feedback; follow other PR links only for specific evidence the scope needs. Ignore unrelated bundled content; leave tracker state unchanged. Return cited answers, decision status (confirmed, proposed, or superseded), blockers, and evidence gaps — cite tickets, comment permalinks, or attachment names and locations; omit raw threads and attachments; mark answers partial if unseen material could change them; stop when the scope is supported.

`git-hosts.md`: include two PR-branch operations. `Align PR branch for editing` — reuse an existing branch that tracks the head repository's `refs/heads/<pr-branch>`; a same-named branch with a different upstream is a collision → check out a distinct branch at the PR head commit, never modify it; require a clean worktree including untracked files; fast-forward a behind branch (behind alone never blocks); report ahead commits — the push publishes them; replay a diverged branch only when the range is linear (`git rebase <pr-head-commit>`; a merge in range → stop; conflict → `git rebase --abort`, stop); after checkout, match a Git remote to the head repository (fork checkouts may add it); verify the PR head commit is an ancestor before edits. `Recover rejected PR push` — on non-fast-forward re-fetch the exact PR head; replay only the commits above the recorded head (`git rebase --onto <new-head> <pr-head-commit>`), linear range only — a merge → stop, conflict → `git rebase --abort`, stop; revalidate, then retry once for the unchanged batch or stop; never force-push.

## Verify and Output

Confirm every generated file exists. Report a summary table: file, tool or reference, configured or not. List files still carrying `<!-- Fill in: ... -->` markers as manual-completion gaps. Do not commit or push.
