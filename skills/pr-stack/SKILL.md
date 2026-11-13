---
name: pr-stack
description: "Use when evaluating whether an existing pull request should become stacked PRs, comparing stacking with keeping one PR or splitting independently, or converting a PR into a stack. Analyze purpose, linked bugs/specs/tickets/cards, project memories, conventions, and deployment; present the assessment and wait for explicit user agreement before conversion."
---

## Required `<project-root>/docs` reads

Resolve `<project-root>` to the PR's project. Shell-read before assessment; may be gitignored — missing → note gap, never invent: `<project-root>/docs/git-hosts.md`; `<project-root>/docs/issue-trackers.md` for linked issues/tickets/cards · `<project-root>/docs/bug-trackers.md` for linked error reports · `<project-root>/docs/doc-lookup.md` when checking current platform support. Memory and code exploration follow `memory-usage` and `context-gathering`.

Assessment is read-only. Until Step 4 agreement, do not edit files, create or switch branches, rewrite commits, push, change PR metadata, or register a stack. Report in chat; do not post the assessment to PRs or tickets without permission.

### 1. Identify and Snapshot the PR

PR URL/number → `<pr-ref>`; extra constraints → `<additional-context>`. Without a reference, use the current branch's PR only when it identifies one open PR unambiguously; otherwise ask.

Load the PR's state, title, body, base/head repositories, branches and SHAs, default branch, commits, complete changed-file list and diff, comments, reviews, checks, and existing stack membership through `git-hosts.md`. Page through truncated results. Record `<original-head>`, `<original-base>`, merge base, and `<pr-url>` as `<snapshot>`. Closed/merged PR, unavailable diff, or ambiguous target → report and stop. Already stacked → assess the existing boundaries; do not create a duplicate stack.

### 2. Establish Purpose and Project Constraints

Invoke `memory-usage` for the **PR's project**, not the skill repository. Read its memory index and relevant architecture, review, release, deployment, and past-incident entries; verify consequential claims against current repository evidence. Invoke `context-gathering` for the changed slice and its dependencies.

Read every directly linked bug, specification, issue, ticket, or card and its relevant comments, acceptance criteria, and attachments. Include references from the PR body, branch name, commits, and review discussions; follow further links when they clarify requirements or dependencies. Search project docs/trackers for missing context when identifiers or the stated purpose suggest related work. Use the matching tracker spec. Reconcile conflicting requirements and separate intended behavior from incidental refactors. Store `<purpose>`, acceptance criteria, dependencies, and source links. Unavailable sources → name the gaps; continue the assessment where evidence suffices, but do not invent requirements.

Inspect project instructions and conventions, repository/module structure, ownership, tests, build configuration, CI workflows, and release/deployment configuration for the affected paths at the recorded PR head/base, not an unrelated checkout. Establish:

- The intended trunk, branch/PR naming and templates, merge method, protection rules, required reviews/checks, and merge queue.
- Whether merging each layer triggers production deployment, a preview, a release, or no deployment; check event/branch/path filters and external deployment configuration where available.
- Compatibility at every prospective merge boundary: schema/data migrations, API consumers, generated artifacts, feature flags, rollout/rollback order, and separate services or packages.
- Git host support and available stack operations, permissions, fork constraints, and effects on existing reviews and collaborators.

Missing deployment or protection evidence stays unknown, not safe by default. **Done when:** the purpose and acceptance criteria have sources, relevant memories/conventions have been reviewed, and constraints or unresolved gaps are explicit.

### 3. Judge Whether Stacking Helps

Read [GitHub stack mechanics](references/github-stacks.md). Verify current host capabilities through `git-hosts.md` or documentation lookup when uncertain; distinguish a registered native stack from ordinary chained PRs.

Map **every changed file and hunk**, including tests, migrations, generated files, configuration, docs, binary assets, renames, and deletions, to coherent concerns. Trace dependencies between concerns and map every acceptance criterion to the concerns that satisfy it. File boundaries and existing commits are evidence, not mandatory split points.

Weigh the alternatives against this PR's evidence:

| Dimension | Favors stacking | Favors keeping one PR or another split |
|-----------|-----------------|----------------------------------------|
| Review | Distinct concerns yield focused diffs and clear reviewer ownership. | Small/cohesive diff; repeated context would obscure the behavior. |
| Dependencies | A linear sequence of meaningful changes builds toward one purpose. | Unrelated concerns can target trunk independently; a forced chain adds delay. |
| Correctness | Each layer builds and passes its relevant checks with its prerequisites below it. | Circular coupling or artificial boundaries leave broken intermediate states. |
| Deployment | Each permitted merge boundary preserves contracts and rollout order. | Partial deployment breaks schema/API compatibility or violates release policy. |
| Delivery cost | Smaller reviews unblock dependent work or expose risky foundations early. | Urgent fix, mature approvals, shared branches, churn, or CI/restacking costs outweigh the benefit. |

Size alone never decides. Keep tests with the behavior they verify; do not create a code-only layer whose required tests arrive later. Prefer the fewest useful layers. A schema → API → UI shape is a candidate only when the actual dependencies and deployment process support it.

Choose `<recommendation>`: **stack**, **keep one PR**, **split independently**, or **insufficient evidence**. Give evidence-backed reasons for and against stacking, including review-context loss, check costs, and operational risk. For a candidate stack, define each layer's purpose, dependencies, files/hunks, acceptance criteria, validation, and deployment/rollback implications. Every original hunk or non-text change belongs to exactly one layer; each layer contains its prerequisites or depends only on lower layers. Flag unrelated scope rather than silently dropping it.

### 4. Present the Assessment and Wait

Report in chat:

- PR link and assessed head/base SHAs; purpose and linked source evidence.
- Recommendation, expected benefit, costs, project-specific constraints, and unresolved gaps; cite repository evidence as `path:line`.
- For a candidate stack, a bottom-to-top table: layer/title, proposed branch and base, concern/files or hunk boundaries, dependencies, checks, and merge/deployment safety. Explain why each boundary helps reviewers.
- Conversion plan: native stack or ordinary chain, which layer retains the original PR and discussion, new PRs, metadata/base changes, and any proposed history rewrites or closures. Warn that retargeting or rewriting may invalidate reviews or approvals. Identify affected shared branches and a recovery plan.

For **keep one PR** or **split independently**, explain why stacking loses and leave the PR unchanged. For **insufficient evidence**, ask for the specific missing information. Do not convert an independent split under stack approval.

For a concrete candidate, ask: **"Do you agree with this assessment and approve converting <pr-url> into the proposed stack?"** Then **STOP and wait**. The initial request, a prior general preference for stacks, silence, or permission to assess is not agreement to this plan. Material plan changes require renewed agreement. Declined → finish unchanged; requested revisions → revise the assessment first.

### 5. Convert Only the Agreed Plan

On explicit agreement, re-read the PR head/base, reviews, and stack membership. Changed since `<snapshot>` → reassess the differences; renew agreement if boundaries, scope, or risks change. Unresolved requirements or merge/deployment safety gaps → resolve before conversion.

Use the approved project workflow through `git-hosts.md`; prepare an isolated workspace through `using-workspaces` when needed, preserving unrelated work. Save recoverable refs to the original commits. Reuse clean commit boundaries where they match the plan; otherwise split mixed commits by hunk into new branches, preserving authorship and the original behavior. Keep the original PR and discussion on the agreed layer when possible.

Create the bottom layer from the agreed trunk and each higher layer from the preceding layer's head. Use `pr-create` for new PRs, forwarding the agreed bases, titles, purpose, ticket references, and layer scope. When native support was agreed, register/link the chain with the supported stack operation; branch targeting alone does not register a native GitHub stack. If native support proves unavailable, disclose ordinary-chain maintenance and CI differences and obtain agreement to that changed plan before proceeding.

Follow the approved plan for existing PR metadata and base changes. Rewrite or force-push an existing branch, close a superseded PR, or delete a branch only when the user explicitly approved that named action; use an expected-head lease for any approved force-push. Never overwrite concurrent updates. Do not merge, enable auto-merge, or deploy as part of conversion.

### 6. Verify and Report

Verify every layer's diff against its immediate base, linear ancestry, base/head repositories, native membership when promised, and required checks. Run the project's relevant build/tests/lint for **each layer with its lower layers**, not only the top. A broken layer requires revising the boundaries, not deferring its checks to a later PR. Compare the cumulative stack diff and final tree with the original PR on a common base: no lost, duplicated, or added behavior. Account separately for upstream changes absent from `<original-head>`, including any trunk movement, before claiming equivalence.

Ensure each PR describes its own scope, links its dependencies and the original context, and explains review/merge order and deployment constraints. Put issue-closing references only on the layer that completes the issue. Recheck the hunk and acceptance-criteria mapping for complete coverage. Wait for checks and address conversion-caused failures; report unavailable checks and pre-existing failures accurately. A blocked or partial conversion stays incomplete: preserve recovery refs and identify what remains; no automatic destructive rollback.

Report the bottom-to-top PR links, the original PR's disposition, validation/equivalence results, and any remaining blockers or required human actions.
