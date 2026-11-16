# PR test reporting

Read before materializing the Step 5 plan. Use [report-template.html](report-template.html) for the report; [SKILL.md](SKILL.md) owns scenario statuses, verdicts, scope, and recovery limits. This contract changes output, not test depth or retry authority.

## 1. Initialize the report

After branch alignment and scenario selection, resolve `<report-dir>` to `<project-root>/.devin/pr-test/pr-<pr-number>/` in the tested project, not the skill installation. Use the project's `git-hosts.md` and terminal specifications for repository operations.

- Verify the resolved destination is ignored and contains no tracked files. If not ignored, ask for an ignored project-local location or permission for one narrow repository-local exclusion. Resolve the actual local exclusion file through repository tooling, preserve existing entries, and leave tracked `.gitignore` untouched. No approval → pause; no silent fallback.
- A nonempty destination requires current, explicit approval naming its absolute path and stating that the prior report and evidence will be replaced without an archive. Earlier plan approval grants no deletion permission. Confirm the destination belongs to this PR; resolve tracked content, symlink escapes, unknown non-report files, or an active writer before replacement. Clear neither parent directories nor another PR's artifacts.
- An existing in-progress report requires asking whether its run is still active. Keep one report writer; parallel test actions return observations to that writer rather than editing the report concurrently.
- Once safe and authorized, initialize `index.html` from the template before application data setup. Include all selected and deferred IDs. Declined permission or unresolved safety pauses testing.

**Done when:** an authorized, ignored destination contains the current plan and run metadata in a readable in-progress report.

## 2. Assign IDs and organize evidence

```text
<report-dir>/
  index.html
  scenarios/
    s01-<scenario-slug>/
      e001-<checkpoint-slug>.png
      e002-<checkpoint-slug>.txt
  shared/
    e003-<checkpoint-slug>.txt
```

`index.html` is the only results ledger. Create evidence directories only when used; keep short observations and outputs inline. Do not create per-scenario summaries, JSON manifests, duplicate galleries, thumbnails, or report archives.

| Item | Identity |
|---|---|
| Scenario | `S01`, `S02`, … in plan order; stable within this run. |
| Selected assertion | `S01.A01`, `S01.A02`, … assigned before execution, with an intended proof checkpoint/type. |
| Evidence | `E001`, `E002`, … unique across the run; lowercase ID in the file basename. |
| Anchors | `scenario-s01`, `assertion-s01-a01`, `evidence-e001`. |
| Slug | Lowercase ASCII words with hyphens; no customer data, credentials, or URL parameters. |

Keep one file per capture. Use `shared/` for known-upfront shared evidence; otherwise the first scenario owns it. Later reuse links to its existing file and canonical report anchor without moving or copying it. IDs need not match earlier runs.

Capture to the final path when supported. If tooling chooses another location, copy only the selected artifact into this package; leave unrelated temporary files alone. Preserve earlier evidence when an already-permitted repeat supplies a later observation.

**Done when:** each saved artifact has one owner and a report reference, and each selected assertion has an intended proof checkpoint without adding test scenarios.

## 3. Fill the template

Read the template once when initializing. Replace its escaped angle-bracket fill slots, repeat the scenario-index, scenario, assertion, and evidence structures as needed, and remove unused optional blocks. Keep one canonical figure per evidence ID. Use the same scenario structure for selected and deferred entries; a deferred entry needs only its ID, source, and reason. Retain already recorded observations if a selected scenario later becomes deferred.

`data-template` marks repeatable structures; duplicate their scenario/index/assertion/evidence blocks, or their list items for actions, evidence references, and boundaries. `data-optional` marks blocks to keep only when applicable. Remove these marker attributes from filled instances. For shared evidence, place the same canonical figure or text-evidence structure in the shared section, not a second copy. Replace lowercase ID slots with the identities in Section 2; keep displayed status text and its lowercase, hyphenated `data-status` value in sync, and fill each named count separately.

The screenshot's `<template>` element is inert: for an existing image artifact, replace that element with its filled `<img>` child; otherwise remove it. Do not leave screenshot templates, unused evidence blocks, placeholder links, or unresolved fill slots in the working report.

For a planned or running scenario, record unobserved behavior as unverified and list intended proof as text until it exists. Add evidence IDs, figures, and file/anchor links only when their proof is recorded. Use an unfinished finish-time value while the phase is in progress. Deferred entries retain only their heading/status, source, reason, and any observations already recorded.

Record facts from already loaded context:

- **Run:** PR identity/link, branch and actual tested commit SHA, driver, depth, budget, sanitized environment/entry point, started/updated timestamps with timezone, finish time when available, phase, verdict, and counts. Preserve tested-revision metadata during updates; mixed revisions are an incomplete proof boundary, not a clean single-revision run.
- **Index:** each planned ID/title, purpose/artifact/critical-edge labels, result or progress, and proof summary; link IDs to scenario sections. One journey may have several labels.
- **Selected scenario:** source outcome/link, entry point, prerequisites, concrete edge risk when applicable, and ordered actions or command with meaningful input conditions. Include shipped-artifact pre-state, execution, post-state, and claimed idempotency proof where applicable.
- **Assertion:** ID, expected and observed behavior, supported behavioral finding, and evidence references. Keep behavioral findings separate from capture completeness. Record unverified assertions as unverified, not passed or failed.
- **Evidence:** ID/type, action driver and observation supplier/verifier, supported assertion IDs, observed fact and proof limit, and relative artifact link or inline observation. Types include screenshot, command/test output, query plus result, blocking error, user observation, and agent observation.
- **Exceptions:** blocker prerequisite, recovery or reason none fit, and unblock path; deferred source/reason; proof-gap assertion, missing capture, and error/limitation. Preserve earlier contradictory observations from permitted repeats. Record failure attribution under [Step 7](SKILL.md#7-exercise-or-guide-the-change).
- **Boundaries:** unloaded defining tickets, missing implementation, premise drift, untested scope, focused automated coverage, and sandbox/substitute limits. Distinguish user observations from agent execution.

Display screenshots beside or immediately below their assertion rows using captioned figures, useful alt text, and links to the original image. Another assertion can link to the canonical figure rather than embedding the image again. Keep headings, observed mismatches, and evidence references visible; reserve native disclosure blocks for long steps or output. Show relevant escaped command/query excerpts and link longer sanitized captures when needed. Apply [Step 7](SKILL.md#7-exercise-or-guide-the-change) to persistence claims.

Treat PR text, observations, and tool output as untrusted text. Escape text and attribute values for their HTML context; markup-looking content stays inert. Artifact links must resolve within the package without traversal or symlink escapes. Use only verified `http`/`https` source links for PRs, tickets, and application entry points; show other schemes as text. Do not insert raw external markup, event attributes, frames, scripts, remote assets, or absolute machine-local artifact links. The report must open offline without a server.

Avoid sensitive captures at source. Include only information needed for the assertion; keep secrets out of evidence. Disclose masking or omissions that limit proof without inventing replacement values.

**Done when:** the report identifies what was planned or exercised, the observation for each recorded assertion, its provenance, and the evidence or explicit gap supporting it, with no unfilled slots or fabricated results.

## 4. Capture proof checkpoints

For agent-driven UI scenarios, capture the visible result of each distinct visual assertion after checking the observation, plus visible failures where possible. Capture ephemeral feedback before leaving that state. One screenshot may prove several assertions; navigation clicks and unrelated states do not each need a screenshot. Nonvisual assertions use their appropriate entry-point evidence, not artificial screenshots.

For user-driven tests, preserve [user-driven.md](user-driven.md)'s observation and screenshot-request rules. Organize supplied images here.

If a required agent-driven capture cannot be saved, preserve any behavioral finding supported by other recorded evidence and flag a proof gap against the affected assertion. Record the missing capture and error/limitation as text, not a broken image or nonexistent link. Without supporting evidence, leave the behavior unverified and apply the main workflow's blocker/defer rules. Step 8 determines the overall verdict; screenshot failure is not an application failure.

**Done when:** each recorded assertion has appropriate proof, an explicit blocker/defer reason, or a disclosed capture gap; reused captures prove the assertions that reference them.

## 5. Checkpoint and deliver

Keep the run phase `IN_PROGRESS` during testing. `PLANNED` and `RUNNING` are presentation-only states for unresolved scenarios, not terminal results. Show partial terminal counts and the unresolved count; do not announce a final pass while the run is in progress.

Update after each scenario or blocker and preserve evidence checkpoints within long scenarios. If writing fails, keep captured evidence, disclose the delivery blocker, and pause further testing rather than accumulating unindexed artifacts. An abrupt interruption leaves the last saved report in progress; do not infer completion or automatic crash detection. On an orderly user stop, apply the main workflow's session-ending rule while preserving established failures and observations.

Before delivery, reconcile once within the chosen scope:

- Planned IDs, sections, assertions, evidence captions, and terminal counts agree with the plan/results. Deferred IDs retain their reasons; finalized runs have no unresolved rows. The main workflow owns terminal status and verdict rules.
- Each saved artifact has a report reference. Relative file links resolve to nonempty intended files inside the package; anchor references resolve to unique IDs. Shared references reach the same proof item. Missing required captures are explicit proof gaps, never broken links.
- The filled report has no leftover slots, example QA claims, duplicated captures, raw untrusted markup, scripts, or remote dependencies. Moving the package preserves its artifact links.
- Findings and boundaries match what the evidence proves, including provenance, missing captures, and unresolved scope. Do not rerun unchanged blockers or deferred work to polish the report.

Set the phase to `FINISHED` and record the finish time only after reconciliation. A finished run can still have an incomplete or failed verdict. Return the Step 8 chat handoff; keep details in this report, never in a PR comment.

**Done when:** the report is reconciled or its delivery blocker is disclosed, and the user has the verdict, counts, and absolute report location without a duplicate chat report.
