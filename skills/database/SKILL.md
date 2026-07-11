---
name: database
description: >
  Use Laravel Boost MCP database tools and the Tinker REPL for database inspection,
  analysis, and operations in the exlink project. Activate when querying the database,
  reading schema, reconciling counts, running EXPLAIN plans, or performing read/write
  operations via Tinker. Do NOT activate for code-level work on models/repositories —
  use Serena there.
---

# Database Analysis & Operations

## Database as Source of Truth

When database tools are available (e.g., `laravel-boost` MCP, Tinker), treat the live database as the source of truth. Schema files, migrations, seeders, factories, and operations describe intent and history; the database records current reality. Inspect the live database before relying on code artifacts for data shape, relationships, constraints, or sample values.

Prefer:
- `database-schema` (or equivalent) for current tables, columns, indexes, foreign keys, views, and routines
- `database-query` for row counts, sample rows, actual values, duplicates, orphans, and `EXPLAIN` plans
- Tinker for Eloquent semantics, scopes, and accessors when the raw schema is not enough

Avoid using migration files, schema dumps, seeders, or operations as proxies for the current database state.

## Tool Inventory

**Laravel Boost MCP (read-only, server: `laravel-boost`)**: Rejects mutating statements.

| Tool | Purpose | Key params |
|------|---------|-----------|
| `database-connections` | List configured connection names | — |
| `database-schema` | Inspect tables, columns, indexes, FKs, views, routines | `summary`, `filter`, `include_column_details`, `include_views`, `include_routines`, `database` |
| `database-query` | Run read-only SQL (`SELECT`, `SHOW`, `EXPLAIN`, `DESCRIBE`) | `query` (required), `database` |

**Tinker (read/write, via kool)**: The only path for mutating via Eloquent. Boots the full app container — models, scopes, gates, company scoping, subscribers fire. Prefer Boost `database-query` for raw SQL (faster, no boot, sandboxed). Use Tinker only for Eloquent semantics or writes.

**Targeting the testing database**: Tests run against a separate `testing` database (`.env.testing`, `DB_DATABASE=testing`), not the local dev DB. When debugging a test failure that depends on seeded data, company IDs, or pre-existing rows, inspect the `testing` DB, not the local dev DB — IDs and seeded rows differ.

- Tinker against testing DB: `kool exec app php artisan tinker --env=testing` (loads `.env.testing`). Do NOT use `kool run artisan tinker` for test-debugging — it hits the local dev DB.
- Boost MCP against testing DB: pass `database: "testing"` to `database-query` and `database-schema`. Without it, Boost hits the default connection's local dev database.

## Discovery Workflow

1. `database-connections` — confirm the default connection (exlink: `mysql` for app, `sqlite` for some CI/test)
2. `database-schema` (`summary: true`) — table names + column types only (cheap overview)
3. `database-schema` (`filter: "<table-substring>"`, `include_column_details: true`) — full metadata (nullable, default, auto-increment, comments). Add `include_views: true` for datatable queries joining views, `include_routines: true` for stored procedures.
4. `database-query` — validate row counts, sample rows, and `EXPLAIN` plans against schema

## Querying Guidelines

- Always `EXPLAIN` before trusting query cost. Confirm index usage, especially on large tables (`feedbacks`, `student_programs`, `placements`, `contacts`).
- Qualify columns when joining. exlink shares many column names (`id`, `company_id`, `created_at`, `deleted_at`, `status`). Prefix every column with its table alias.
- Respect soft deletes. Raw SQL must add `AND <alias>.deleted_at IS NULL` manually — Boost won't. Mirror Eloquent `joinRelationship()` semantics before writing JOINs.
- Company scoping is not applied by Boost. `database-query` runs literal SQL with no global scopes. To match a Filament widget/datatable, replicate its `queryAll`/`queryScope` filters explicitly (status exclusions, partial-access scoping, precedence unions). See *Reconciling Counts* below.
- Limit large results. Add `LIMIT` to exploratory `SELECT`s. Boost returns full result sets; unbounded joins over `placements` flood the response.

## Reconciling Counts (widget vs. raw SQL)

When a raw SQL count disagrees with a UI widget:

1. Read the widget's query path with Serena (`find_symbol` on the repository's `getDatatableQuery` / `getStDatatable`).
2. Reproduce every JOIN and WHERE in `database-query` `SELECT COUNT(*)`.
3. If counts still differ, the gap is almost always in `queryAll`/`queryScope` — company precedence unions, partial-access filters, or cancelled-status exclusions the raw SQL omits.
4. Add filters one at a time until counts match. The missing filter is the root cause.

Never "fix" the widget to match the raw count without identifying the filter gap — widget filters are intentional.

## When to Use Tinker Instead of Boost

| Need | Tool |
|------|------|
| List connections | Boost `database-connections` |
| Read schema / columns / indexes | Boost `database-schema` |
| Raw `SELECT` / `EXPLAIN` / `SHOW` | Boost `database-query` |
| Count rows matching widget, with Eloquent scopes | Tinker (`Model::count()` with scopes) |
| Inspect accessor, cast, or relationship output | Tinker |
| Mutate data (insert/update/delete) | Tinker — only with explicit user approval |
| Trigger model events / subscribers | Tinker |
| Resolve contract/repository (`Repository::resolve()`) | Tinker |

## Tinker Safety Rules

- Never write without explicit user confirmation for that specific action, including `save()`, `update()`, `delete()`, `forceDelete()`, `truncate()`, and seeder/operation invocation. State what will change and wait.
- Prefer `factory()` or a transaction for experiments. Wrap exploratory writes in `DB::transaction(function () { ... DB::rollback(); })`.
- Company scoping applies in Tinker. Repository/contract queries scope to the current company. For unscoped data, call `queryAll()` or use `without*` scopes deliberately — this bypasses access controls.
- Subscribers fire. Creating/updating/deleting models triggers `Modules\<X>\Subscribers\*` listeners, which may enqueue jobs, write logs, or sync services. Use raw `DB::table()` writes only when you need to bypass events — document why.
- Never commit secrets or paste `.env` values into Tinker snippets you share.

## Pitfalls

- `database-schema` `summary` omits indexes and FKs; re-call with `filter` + `include_column_details` for real work.
- Views and routines are hidden by default; set `include_views` / `include_routines` when queries reference them.
- Tinker boots the full app — slow startup (~seconds). Batch questions into one session.

## Anti-Patterns

- Using SQL schema dump (`database/schema/mysql-schema.sql`) to investigate the current DB — it is a historical/testing snapshot and may be stale. Use `database-schema` (Boost MCP).
- Using `grep`/`read` to find a model's table or fillable columns — use `database-schema` with `filter`.
- Writing raw `UPDATE`/`DELETE` SQL expecting Boost to run it — Boost is read-only; use Tinker.
- Relying on seeders, migrations, factories, or operations to infer current data shape, relationships, or valid values — inspect the live database instead.
- Assuming a migration ran exactly as written — verify the resulting schema and sample rows with `database-schema` / `database-query`.

- Don't echo `EXPLAIN` output raw. Report: "Uses index X, scans N rows."

