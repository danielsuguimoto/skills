---
name: browser-automation
description: Use for browser automation (QA, element interaction, screencapture/snapshot, network capture) or to work across the user's logged-in accounts, apps, memory, and browsing history.
---

## Required `<project-root>/docs` reads

Shell-read before any browser automation action; may be gitignored — missing → note gap, never invent: `<project-root>/docs/browser-automation.md`

# Browser Automation

## Choose the Surface

Choose the configured direct interaction or delegation surface per `browser-automation.md`. Delegate only when the `subagents` gates permit it. Respect the tool's session and tab lifecycle.

## Open Browser Tabs

When the user names a current page or tab, inspect open tabs and attach to the relevant one. Open a new tab only when needed or requested.

## Snapshot

Use the configured snapshot or accessibility tree as the primary page read. Use fresh references after state changes; do not guess selectors, URLs, or page content. If the page is still changing, wait and read again; use screenshots when the structured view is insufficient.

## Navigation and Actions

- Batch an action and its verification when supported; split dependent actions.
- Confirm each action from fresh page state. Unexpected state → check for a missed or stale target before inferring site requirements.
- Follow the configured tool's wait, scrolling, tab creation, and cleanup semantics.

## Forms, Autofill, and Login

Prefer available autofill for credentials and forms. If incomplete, inspect the page and continue manually; ask the user when access or a decision requires them.

## Downloads

Use the configured download operation for buttons, blob URLs, redirects, or POST-backed downloads. A direct request is limited to trusted same-origin or direct-download GET/HEAD URLs; do not forward credentials across origins. Verify the downloaded file within its session and report only confirmed facts.
