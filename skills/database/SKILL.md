---
name: database
description: "Use database inspection tools for schema, queries, reconciling counts, EXPLAIN plans, read/write operations. Not for code-level model/repository work — use code navigation tools there."
---

## Required `<project-root>/docs` reads

Shell-read before any database query, schema inspection, or read/write operation; may be gitignored — missing → note gap, never invent: `<project-root>/docs/code-navigation.md`, `<project-root>/docs/database-tools.md`

## Source of Truth

Treat the live database as source of truth — schema files, migrations, seeders, factories, and operations describe intent; the database records reality. Inspect live state before relying on code artifacts for data shape, relationships, constraints, or sample values.

Prefer (`<project-root>/docs/database-tools.md`): schema-inspection tool (tables, columns, indexes, FKs, views, routines) · read-only query tool (counts, samples, duplicates, orphans, `EXPLAIN`) · REPL for Eloquent semantics.

## Tool Inventory

**Read-only tools** (reject mutating statements): list connections · inspect schema (`summary`, `filter`, `include_column_details`, `include_views`, `include_routines`, `database`) · read-only query (`SELECT`/`SHOW`/`EXPLAIN`/`DESCRIBE`; `query` required).

**REPL (read/write)** — boots the full app container: models, scopes, gates, company scoping, subscribers fire.

**Testing database**: test failures depending on seeded data, company IDs, or pre-existing rows → inspect the testing DB, not local dev — IDs and seeded rows differ. Target it via `<project-root>/docs/database-tools.md`; otherwise both tools hit the default connection's local dev DB.

## Discovery Workflow

1. List connections — confirm the default.
2. Schema `summary: true` — table names + column types (cheap overview).
3. Schema `filter: "<table-substring>"`, `include_column_details: true` — full metadata. Add `include_views`/`include_routines` for datatable queries joining views or stored procedures.
4. Query tool — validate counts, sample rows, `EXPLAIN` plans.

## Querying Guidelines

- `EXPLAIN` before trusting query cost — confirm index usage, especially on large tables.
- Qualify every joined column with its table alias — many tables share `id`, `company_id`, `created_at`, `deleted_at`, `status`.
- Respect soft deletes: raw SQL needs explicit `AND <alias>.deleted_at IS NULL` — read-only tools apply no global scopes. Mirror Eloquent `joinRelationship()` semantics before JOINs.
- Company scoping is not applied — replicate the widget's `queryAll`/`queryScope` filters explicitly (status exclusions, partial-access scoping, precedence unions). See *Reconciling Counts*.
- `LIMIT` exploratory `SELECT`s — unbounded joins over large tables flood the response.
- Report `EXPLAIN` as prose ("Uses index X, scans N rows"), never raw output.

## Reconciling Counts (widget vs. raw SQL)

1. Read the widget's query path via code navigation (`getDatatableQuery`/`getStDatatable`).
2. Reproduce every JOIN and WHERE with `SELECT COUNT(*)`.
3. Still differs → the gap is almost always in `queryAll`/`queryScope` (precedence unions, partial-access filters, cancelled-status exclusions raw SQL omits); add filters one at a time until counts match — the missing one is the root cause.

Never "fix" the widget to match the raw count — widget filters are intentional.

## Query Tool vs. REPL

| Need | Tool |
|------|------|
| Connections, schema, raw `SELECT`/`EXPLAIN`/`SHOW` | Read-only query tool |
| Widget-matching counts with Eloquent scopes | REPL (`Model::count()` with scopes) |
| Accessor, cast, or relationship output | REPL |
| Mutate data | REPL — explicit user approval only |
| Model events/subscribers, contract resolution (`Repository::resolve()`) | REPL |

## REPL Safety Rules

- No writes without explicit user confirmation for that action — `save()`, `update()`, `delete()`, `forceDelete()`, `truncate()`, seeder/operation invocation. State what will change and wait.
- Experiments: `factory()` or `DB::transaction(function () { ... DB::rollback(); })`.
- Company scoping applies — repository/contract queries scope to the current company; `queryAll()`/`without*` scopes bypass access controls deliberately.
- Subscribers fire on model writes — may enqueue jobs, write logs, sync services. Raw `DB::table()` bypasses events — document why when used.
- REPL boots slowly (~seconds) — batch questions into one session.
