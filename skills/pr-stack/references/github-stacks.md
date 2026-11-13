# GitHub Stacked PRs

Research basis: official GitHub documentation. Check current support before relying on host-specific behavior; an ordinary chain and a registered native stack have different guarantees.

## Model and Review Benefits

A stack is an ordered series of dependent PRs in one repository. Its bottom PR targets the trunk, usually the default branch but possibly a release branch. Each higher PR targets the head branch immediately below it. Reviewers see each layer's incremental diff rather than the cumulative feature.

This helps when work has meaningful dependency boundaries: a reviewer can assess a foundation separately while development continues above it. Separate ownership can allow parallel reviews. The gain depends on coherent layers; reducing line count while fragmenting one behavioral change increases review burden. Independent work does not need a dependency chain.

## Native Stacks Versus Ordinary Chains

| Concern | Registered native GitHub stack | Ordinary dependent PRs |
|---------|--------------------------------|------------------------|
| Registration | Link compatible PRs into a stack; GitHub exposes stack membership and a stack map. | Setting base branches creates a chain but does not itself register a native stack. |
| Rules and CI | GitHub evaluates trunk protections, reviews, and required checks for every layer; Actions PR branch filters use the stack trunk. | Rules and workflow filters follow ordinary base-branch behavior; upper layers may miss trunk-only checks. |
| Updates | Native operations support cascading rebases; merging the bottom automatically retargets/rebases the remaining stack. | Authors must verify retargeting and synchronize branches; squash/rebase merges can change ancestry and expose duplicate diffs. |
| Merge | History between layers must be linear. A selected PR and all layers below it can land together; remaining upper layers stay open. | Merge/order/synchronization behavior needs an explicit project workflow. |

Native stacks require all branches in the same repository; cross-fork stacks are unsupported. Existing PRs with compatible base/head relationships can be linked into a stack. Verify repository permissions and tool support rather than assuming availability from the provider name.

GitHub supports merge commit, squash, and rebase methods for native stacks and supports merge queues. Protections still apply. Each selected PR and its lower layers must satisfy requirements before merging. Platform merge support does not establish application deployment safety: inspect the project's real deployment triggers and whether it permits partial merges.

Changing an existing PR's base can remove commits from its timeline and make review comments outdated. Do not promise that approvals survive conversion; check repository settings and the actual changes. Native rebases that preserve the diff can retain approvals, but splitting or changing a layer's diff is a different operation.

## Sources

- [About stacked pull requests](https://docs.github.com/en/pull-requests/get-started/about-stacked-prs) — model, review benefits, surfaces, and limitations.
- [Stacked pull requests reference](https://docs.github.com/en/pull-requests/reference/stacked-pull-requests) — protections, Actions, linear history, merge methods, and merge queues.
- [Creating stacked pull requests](https://docs.github.com/en/pull-requests/how-tos/create-pull-requests/creating-stacked-pull-requests) — branch relationships, native registration, and linking existing PRs.
- [Stack code changes in pull requests](https://docs.github.com/en/pull-requests/tutorials/stack-code-changes-in-pull-requests) — coherent layers, dependency order, per-layer review and validation.
- [Changing the base branch of a pull request](https://docs.github.com/en/pull-requests/how-tos/create-pull-requests/changing-the-base-branch-of-a-pull-request) — timeline and review-comment effects.
