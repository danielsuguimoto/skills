---
name: database
description: >
  Use database inspection tools for schema, queries, and operations. Activate when
  querying the database, reading schema, reconciling counts, running EXPLAIN plans,
  or performing read/write operations. Do NOT activate for code-level work on
  models/repositories — use code navigation tools there.
---

# Database Analysis & Operations

## Database as Source of Truth

When database tools are available (see `/docs/database-tools.md` in the project root), treat the live database as the source of truth. Schema files, migrations, seeders, factories, and operations describe intent and history; the database records current reality. Inspect the live database before relying on code artifacts for data shape, relationships, constraints, or sample values.

Prefer:
- Schema inspection tool for current tables, columns, indexes, foreign keys, views, and routines (see `/docs/database-tools.md` in the project root)
- Query tool for row counts, sample rows, actual values, duplicates, orphans, and `EXPLAIN` plans (see `/docs/database-tools.md` in the project root)
- REPL for Eloquent semantics, scopes, and accessors when the raw schema is not enough (see `/docs/database-tools.md` in the project root)

Avoid using migration files, schema dumps, seeders, or operations as proxies for the current database state.

## Tool Inventory

**Database tools (read-only, see `/docs/database-tools.md` in the project root)**: Rejects mutating statements.

| Operation | Purpose | Key params |
|-----------|---------|-----------|
| List connections | List configured connection names | — |
| Inspect schema | Inspect tables, columns, indexes, FKs, views, routines | `summary`, `filter`, `include_column_details`, `include_views`, `include_routines`, `database` |
| Run read-only query | Run read-only SQL (`SELECT`, `SHOW`, `EXPLAIN`, `DESCRIBE`) | `query` (required), `database` |

**REPL (read/write, see `/docs/database-tools.md` in the project root)**: The only path for mutating via Eloquent. Boots the full app container — models, scopes, gates, company scoping, subscribers fire. Prefer the query tool for raw SQL (faster, no boot, sandboxed). Use the REPL only for Eloquent semantics or writes.

**Targeting the testing database**: Tests may run against a separate testing database (see `/docs/database-tools.md` in the project root for environment targeting). When debugging a test failure that depends on seeded data, company IDs, or pre-existing rows, inspect the testing DB, not the local dev DB — IDs and seeded rows differ.

- REPL with testing environment (see `/docs/database-tools.md` in the project root). Do NOT use the REPL with default environment for test-debugging — it hits the local dev DB.
- Query tool against testing DB: target the testing database per `/docs/database-tools.md` in the project root. Without it, the query tool hits the default connection's local dev database.

## Discovery Workflow

1. List connections — confirm the default connection (see `/docs/database-tools.md` in the project root)
2. Schema inspection (`summary: true`) — table names + column types only (cheap overview)
3. Schema inspection (`filter: "<table-substring>"`, `include_column_details: true`) — full metadata (nullable, default, auto-increment, comments). Add `include_views: true` for datatable queries joining views, `include_routines: true` for stored procedures.
4. Query tool — validate row counts, sample rows, and `EXPLAIN` plans against schema

## Querying Guidelines

- Always `EXPLAIN` before trusting query cost. Confirm index usage, especially on large tables (`feedbacks`, `student_programs`, `placements`, `contacts`).
- Qualify columns when joining. Many projects share column names (`id`, `company_id`, `created_at`, `deleted_at`, `status`). Prefix every column with its table alias.
- Respect soft deletes. Raw SQL must add `AND <alias>.deleted_at IS NULL` manually — read-only query tools won't apply global scopes. Mirror Eloquent `joinRelationship()` semantics before writing JOINs.
- Company scoping is not applied by read-only query tools. The query tool runs literal SQL with no global scopes. To match a Filament widget/datatable, replicate its `queryAll`/`queryScope` filters explicitly (status exclusions, partial-access scoping, precedence unions). See *Reconciling Counts* below.
- Limit large results. Add `LIMIT` to exploratory `SELECT`s. Query tools return full result sets; unbounded joins over large tables flood the response.

## Reconciling Counts (widget vs. raw SQL)

When a raw SQL count disagrees with a UI widget:

1. Read the widget's query path with the code navigation tool (see `/docs/code-navigation.md` in the project root) — find the repository's `getDatatableQuery` / `getStDatatable`.
2. Reproduce every JOIN and WHERE with the query tool `SELECT COUNT(*)` (see `/docs/database-tools.md` in the project root).
3. If counts still differ, the gap is almost always in `queryAll`/`queryScope` — company precedence unions, partial-access filters, or cancelled-status exclusions the raw SQL omits.
4. Add filters one at a time until counts match. The missing filter is the root cause.

Never "fix" the widget to match the raw count without identifying the filter gap — widget filters are intentional.

## When to Use the REPL Instead of Read-Only Query Tools

| Need | Tool |
|------|------|
| List connections | Read-only query tool — list connections |
| Read schema / columns / indexes | Read-only query tool — schema inspection |
| Raw `SELECT` / `EXPLAIN` / `SHOW` | Read-only query tool — run query |
| Count rows matching widget, with Eloquent scopes | REPL (`Model::count()` with scopes) |
| Inspect accessor, cast, or relationship output | REPL |
| Mutate data (insert/update/delete) | REPL — only with explicit user approval |
| Trigger model events / subscribers | REPL |
| Resolve contract/repository (`Repository::resolve()`) | REPL |

## REPL Safety Rules

- Never write without explicit user confirmation for that specific action, including `save()`, `update()`, `delete()`, `forceDelete()`, `truncate()`, and seeder/operation invocation. State what will change and wait.
- Prefer `factory()` or a transaction for experiments. Wrap exploratory writes in `DB::transaction(function () { ... DB::rollback(); })`.
- Company scoping applies in the REPL. Repository/contract queries scope to the current company. For unscoped data, call `queryAll()` or use `without*` scopes deliberately — this bypasses access controls.
- Subscribers fire. Creating/updating/deleting models triggers `Modules\<X>\Subscribers\*` listeners, which may enqueue jobs, write logs, or sync services. Use raw `DB::table()` writes only when you need to bypass events — document why.
- Never commit secrets or paste `.env` values into REPL snippets you share.

## Pitfalls

- Schema inspection `summary` omits indexes and FKs; re-call with `filter` + `include_column_details` for real work.
- Views and routines are hidden by default; set `include_views` / `include_routines` when queries reference them.
- The REPL boots the full app — slow startup (~seconds). Batch questions into one session.

## Anti-Patterns

- Using SQL schema dumps to investigate the current DB — they are historical/testing snapshots and may be stale. Use the schema inspection tool (see `/docs/database-tools.md` in the project root).
- Using `grep`/`read` to find a model's table or fillable columns — use the schema inspection tool with `filter`.
- Writing raw `UPDATE`/`DELETE` SQL expecting read-only query tools to run it — they are read-only; use the REPL.
- Relying on seeders, migrations, factories, or operations to infer current data shape, relationships, or valid values — inspect the live database instead.
- Assuming a migration ran exactly as written — verify the resulting schema and sample rows with the schema inspection tool / query tool (see `/docs/database-tools.md` in the project root).

- Don't echo `EXPLAIN` output raw. Report: "Uses index X, scans N rows."

