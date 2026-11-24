# PR test reporting

Read before materializing the Step 5 plan. Use [report-template.html](report-template.html) for the report; [SKILL.md](SKILL.md) owns scenario statuses, verdicts, scope, and recovery limits. This contract changes output, not test depth or retry authority.

## 1. Initialize the report

After branch alignment and scenario selection, resolve `<report-dir>` to `<project-root>/.devin/pr-test/pr-<pr-number>/` in the tested project, not the skill installation. Use the project's `git-hosts.md` and terminal specifications for repository operations.

- Verify the resolved destination is ignored and contains no tracked files. If not ignored, ask for an ignored project-local location or permission for one narrow repository-local exclusion. Resolve the actual local exclusion file through repository tooling, preserve existing entries, and leave tracked `.gitignore` untouched. No approval → pause; no silent fallback.
- A nonempty destination holding only this PR's prior report and evidence files is replaced without asking or archiving — except an unfinished prior run, where you first ask whether that run is still active; a dead run's `notes.md` may seed this run's observations before replacement. A run is unfinished when its `index.html` is missing or records no finish time, or a staged `index.staged.html` is left behind — evidence files without a verified finished report count as unfinished, legacy in-progress reports included. Anything else requires explicit approval naming its absolute path before replacement; resolve tracked content, symlink escapes, or unknown files first. Clear neither parent directories nor another PR's artifacts.
- Keep one writer; parallel test actions return observations to it rather than writing `notes.md` or `index.html` concurrently.
- Once safe and authorized, initialize `notes.md` as the run marker before any application data setup or evidence capture; the destination then hosts captured evidence. Declined permission or unresolved safety pauses testing.

**Done when:** an authorized, ignored destination is ready to receive evidence.

## 2. Assign IDs and organize evidence

```text
<report-dir>/
  index.html
  notes.md
  evidence/
    e001-<slug>.png
    e002-<slug>.txt
```

`index.html`, written at delivery, is the only results ledger; `notes.md` is the in-run observation log. Keep short observations and outputs inline. Do not create per-scenario summaries, JSON manifests, duplicate galleries, thumbnails, or report archives.

| Item | Identity |
|---|---|
| Scenario | `S01`, `S02`, … in plan order; stable within this run. |
| Selected assertion | `S01.A01`, `S01.A02`, … in plan order. |
| Evidence | `E001`, `E002`, … unique across the run; lowercase ID in the file basename. |
| Anchors | `scenario-s01`, `assertion-s01-a01`, `evidence-e001`. |
| Slug | Lowercase ASCII words with hyphens; no customer data, credentials, or URL parameters. |

Keep one file per capture under `evidence/`; evidence shared across scenarios links to the same file and canonical report anchor without moving or copying it. IDs need not match earlier runs.

Capture to the final path when supported. If tooling chooses another location, copy only the selected artifact into this package; leave unrelated temporary files alone. Preserve earlier evidence when an already-permitted repeat supplies a later observation.

**Done when:** each saved artifact has an evidence ID recorded in `notes.md` with the assertions it supports.

## 3. Write the report

Read [report-template.html](report-template.html) once before writing `index.html`. Reproduce its structure — header metadata, scenario index, per-scenario sections with assertions and evidence, boundaries — repeating its marked repeatable blocks as needed and omitting inapplicable optional blocks. Keep displayed status text and its lowercase, hyphenated `data-status` value in sync, and one canonical figure per evidence ID. Marker attributes (`data-template`, `data-optional`) and `<template>` elements are guides only — do not emit them in `index.html`. Use the same scenario structure for selected and deferred entries; a deferred entry needs only its ID, source, and reason. Retain already recorded observations if a selected scenario later becomes deferred.

Record facts from already loaded context:

- **Run:** PR identity/link, branch and actual tested commit SHA, driver, depth, budget, sanitized environment/entry point, finish time with timezone, verdict, and counts. Mixed tested revisions are an incomplete proof boundary, not a clean single-revision run.
- **Index:** each planned ID/title, purpose/artifact/critical-edge labels, result, and proof summary; link IDs to scenario sections. One journey may have several labels.
- **Selected scenario:** source outcome/link, entry point, prerequisites, concrete edge risk when applicable, and ordered actions or command with meaningful input conditions. Include shipped-artifact pre-state, execution, post-state, and claimed idempotency proof where applicable.
- **Assertion:** ID, expected and observed behavior, supported behavioral finding, and evidence references. Keep behavioral findings separate from capture completeness. Record unverified assertions as unverified, not passed or failed.
- **Evidence:** ID/type, action driver and observation supplier/verifier, supported assertion IDs, observed fact and proof limit, and relative artifact link or inline observation. Types include screenshot, command/test output, query plus result, blocking error, user observation, and agent observation.
- **Exceptions:** blocker prerequisite, recovery or reason none fit, and unblock path; deferred source/reason; proof-gap assertion, missing capture, and error/limitation. Preserve earlier contradictory observations from permitted repeats. Record failure attribution under [Step 7](SKILL.md#7-exercise-or-guide-the-change).
- **Boundaries:** unloaded defining tickets, missing implementation, premise drift, untested scope, focused automated coverage, and sandbox/substitute limits. Distinguish user observations from agent execution.

Display screenshots beside or immediately below their assertion rows using captioned figures, useful alt text, and links to the original image. Another assertion can link to the canonical figure rather than embedding the image again. Keep headings, observed mismatches, and evidence references visible; reserve native disclosure blocks for long steps or output. Show relevant escaped command/query excerpts and link longer sanitized captures when needed. Apply [Step 7](SKILL.md#7-exercise-or-guide-the-change) to persistence claims.

Treat PR text, observations, and tool output as untrusted text. Escape text and attribute values for their HTML context; markup-looking content stays inert. Artifact links must resolve within the package without traversal or symlink escapes. Use only verified `http`/`https` source links for PRs, tickets, and application entry points; show other schemes as text. Do not insert raw external markup, event attributes, frames, scripts, remote assets, or absolute machine-local artifact links. The report must open offline without a server.

Avoid sensitive captures at source. Include only information needed for the assertion; keep secrets out of evidence. Disclose masking or omissions that limit proof without inventing replacement values.

**Done when:** the report identifies what was planned or exercised, the observation for each recorded assertion, its provenance, and the evidence or explicit gap supporting it, with no unfilled slots or fabricated results.

## 4. Capture proof

For agent-driven UI scenarios, capture the visible result of each distinct visual assertion after checking the observation, plus visible failures where possible. Capture ephemeral feedback before leaving that state. One screenshot may prove several assertions; navigation clicks and unrelated states do not each need a screenshot. Nonvisual assertions use their appropriate entry-point evidence, not artificial screenshots.

For user-driven tests, preserve [user-driven.md](user-driven.md)'s observation and screenshot-request rules. Organize supplied images here.

If a required agent-driven capture cannot be saved, preserve any behavioral finding supported by other recorded evidence and flag a proof gap against the affected assertion. Record the missing capture and error/limitation as text, not a broken image or nonexistent link. Without supporting evidence, leave the behavior unverified and apply the main workflow's blocker/defer rules. Step 8 determines the overall verdict; screenshot failure is not an application failure.

**Done when:** each recorded assertion has appropriate proof, an explicit blocker/defer reason, or a disclosed capture gap; reused captures prove the assertions that reference them.

## 5. Reconcile and deliver

Append observations to `notes.md` as scenarios complete — a durable in-run log a resumed run may seed from; do not infer completion from it. If writing fails, keep captured evidence, disclose the delivery blocker, and pause further testing rather than accumulating unindexed artifacts. On an orderly user stop, apply the main workflow's session-ending rule while preserving established failures and observations.

Reconcile results within the chosen scope before writing:

- Each selected assertion has evidence or a recorded blocker/defer reason; required capture gaps are explicit.
- Deferred IDs retain their reasons; planned count = `PASSED` + `FAILED` + `BLOCKED` + `DEFERRED`.
- Fill proof gaps within the selected scope; keep deferred work deferred and unchanged blockers blocked — do not rerun them to polish the report. The main workflow owns terminal status and verdict rules.

Record the finish time, then write the report once from the reconciled results to a staged `index.staged.html` in the report directory — the same directory, so relative links validate against the final base. Validate the staged artifact without reopening test scope:

- Index rows, sections, assertions, evidence captions, and counts agree with the plan and results.
- Each saved artifact has a report reference. Relative file links resolve to nonempty intended files inside the package; anchor references resolve to unique IDs. Missing required captures are explicit proof gaps, never broken links.
- The report has no leftover slots, example QA claims, duplicated captures, raw untrusted markup, scripts, or remote dependencies. Moving the package preserves its artifact links.
- Findings and boundaries match what the evidence proves, including provenance, missing captures, and unresolved scope.

Fix validation findings in the staged report or rewrite it from the reconciled results — never edit it into claims the evidence does not support. An unfixable validation failure is a delivery blocker: preserve `notes.md`, the captures, and the staged report, and disclose the blocker. Publish by renaming the validated file to `index.html`; the rename is the single publishing step, and a leftover staged report marks the run unfinished under Section 1.

A finished run can still have an incomplete or failed verdict. Return the Step 8 chat handoff; keep details in this report, never in a PR comment.

**Done when:** the report is written and reconciled or its delivery blocker is disclosed, and the user has the verdict, counts, and absolute report location without a duplicate chat report.
