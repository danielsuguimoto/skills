---
name: browser-automation
description: Use for browser automation (QA, element interaction, screencapture/snapshot, network capture) or to work across the user's logged-in accounts, apps, memory, and browsing history.
---

## Required `<project-root>/docs` reads

Shell-read before any browser automation action; may be gitignored — missing → note gap, never invent: `<project-root>/docs/browser-automation.md`

# Browser Automation

## Choose the Surface

- **Browser exec** (autonomous agent): spawns an agent session like a subagent — for whole-task delegation across logged-in accounts, apps, memory, browsing history. Poll and update the user every ~60s.
- **Browser REPL**: persistent ES2023+ JS REPL with Playwright-compatible interaction — for direct evidence, downloads, screenshots, exact verification, deterministic UI steps, sensitive logged-in work. Top-level `const`/`let` persist — use fresh variable names.

Both open ephemeral sessions — use interactive PTY for CLI commands; sessions die when the CLI exits. Inspect current CLI usage first (`browser-automation.md`).

## REPL Globals

`page` (Playwright-like `Page`), `tabs`, `listBrowserTabs()`, `attachBrowserTab(targetId)`, `attachActiveBrowserTab()`, `getTabByTargetId(targetId)`, `openTab(url)`, `closeTab(tab)`, `snapshot(page, options?)` → `{ tree, diff }`, `annotatedScreenshot(page)`, `page.screenshot()`, `page.pdf(options?)` (save under `./artifacts/`), `fetch(url)` (cookie-bearing HTTP; same-origin/trusted GET/HEAD only), `fs`, `path`, `Buffer`, `sleep`, `display`, `pwd`. `console.log()` returns values.

## Open Browser Tabs

`browser repl` starts neutral — `page` is not necessarily the user's current tab. User mentions a current/open page or tab → inspect `listBrowserTabs()` (targetId/active/title/url) first; `attachActiveBrowserTab()` for the active page or `attachBrowserTab(targetId)` for a specific one; then read with `snapshot(page, { interactive: true })`. `openTab()` only when no relevant tab exists or the user asks.

## Snapshot

ALWAYS `snapshot()` as the primary page read — compact accessibility tree with unique ref IDs (`e12`, `f1e1`), page title, URL, child-iframe contents, off-viewport elements. Options: `interactive`, `showHidden`, `ref` (e.g. `"e31"`), `selector` (CSS, e.g. `[role="dialog"]` — not `"dialog"`).

- Ref IDs are virtual locators, not DOM properties — safe for `page.locator('e31')`; NEVER mix into CSS selectors.
- Each snapshot invalidates earlier ref IDs — re-snapshot after each action; save as `const s1`, `const s2`.
- Start with `tree`; after an action, ALWAYS print `diff`.
- NEVER guess ref IDs, selectors, page content, or snapshot size; NEVER truncate with `substring()`/`slice()`/`split()`.

**Reading escalation:** `snapshot(page, { interactive: true })` → `snapshot(page)` → wait + snapshot if still changing → `annotatedScreenshot(page)` (boxes + ref IDs) or `page.screenshot()` (raw). Avoid `page.content()`/`page.evaluate()` unless you know the exact selector.

## Navigation and Actions

- Playwright APIs through `page`. ALWAYS `openTab()`/`closeTab()` for tabs — NEVER `page.context().newPage()`/`page.close()` (memory leaks).
- NEVER guess URLs (well-known destinations like Google/YouTube excepted). Locator actions with ref IDs over `page.evaluate()`.
- Pack action + snapshot in one call when the next step doesn't depend on new state; split when it does.
- Action is unconfirmed until a fresh snapshot shows expected state; unexpected → suspect missed/stale/wrong-target action before inferring site requirements. State-changing results are evidence — recheck only on concrete contradiction or stale snapshot.
- `openTab()`/`click()` already wait for interactivity/DOM stability — `sleep()` only when a snapshot shows the page still transitioning.
- No scroll needed — snapshot includes off-screen elements; click scrolls to targets.

## Forms, Autofill, and Login

Prefer autofill paths for autofillable forms (ID/PW, email, payment, address). Autofill incomplete → fresh snapshot, continue manually. **ASK USER AS LAST RESORT** when stuck.

## Downloads

`fetch()` only for same-origin or trusted direct-download GET/HEAD URLs on the current page — no mutations, cross-origin credential forwarding, or unverified page-text URLs.
```js
await fs.mkdir("./artifacts", { recursive: true });
const href1 = new URL(downloadUrl, page.url()).href;
const res1 = await fetch(href1);
if (!res1.ok) throw new Error(`download failed: ${res1.status}`);
await fs.writeFile("./artifacts/download.pdf", Buffer.from(await res1.arrayBuffer()));
```
Download buttons, blob URLs, redirects, POST-backed → browser download handling. Verify `download.path()`; `download.saveAs("./artifacts/name.ext")` only when an artifacts copy is needed.
```js
const downloadPromise = page.waitForEvent('download');
await page.locator('button.export').click();
const download = await downloadPromise;
console.log({ filename: download.suggestedFilename(), downloadPath: await download.path(), size: (await fs.stat(await download.path())).size });
```
`fs` cannot browse real `~/Downloads`; after `download.path()` the file is readable in-session. One-shot `browser repl "..."` → verify inside the same command. Extract PDF/document facts via local document tools; report only confirmed facts.
