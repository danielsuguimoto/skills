---
name: browser-automation
description: "Automate browser QA, page interaction, screenshots, snapshots, network capture, or work across signed-in accounts, apps, memory, and browsing history."
---

## Required `<project-root>/docs` reads

Shell-read before any browser automation action (reuse session reads): `<project-root>/docs/browser-automation.md`. If a spec is missing, report the gap and use a documented fallback.

# Browser Automation

## Choose the Surface

Choose the configured direct interaction or delegation surface per `browser-automation.md`. Delegate only when the `subagents` gates permit it. Respect the tool's session and tab lifecycle. Tabs in one browser context share cookies and storage — each parallel delegated actor needs a separate context, or a separate profile where the tool has no context support.

## Open Browser Tabs

When the user names a current page or tab, inspect open tabs and attach to the relevant one. Open a new tab only when needed or requested.

## Snapshot

Use the configured snapshot or accessibility tree as the primary page read. Use fresh references after state changes; do not guess selectors, URLs, or page content. If the page is still changing, wait and read again; use screenshots when the structured view is insufficient.

## Navigation and Actions

- Batch predictable actions and verify their combined outcome at a meaningful checkpoint. Confirm consequential actions before proceeding; split actions that depend on an unverified result.
- Use fresh references when state changes invalidate them. Unexpected state → check for a missed or stale target before inferring site requirements.
- Follow the configured tool's wait, scrolling, tab creation, and cleanup semantics.

## Forms, Autofill, and Login

Prefer available autofill for credentials and forms. If incomplete, inspect the page and continue manually; ask the user when access or a decision requires them.

## Downloads

Use the configured download operation for buttons, blob URLs, redirects, or POST-backed downloads. A direct request is limited to trusted same-origin or direct-download GET/HEAD URLs; do not forward credentials across origins. Verify the downloaded file within its session and report only confirmed facts.
