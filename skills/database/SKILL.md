---
name: database
description: "Inspect live schemas, query data, reconcile counts, inspect execution plans, or write data. Use code navigation for model or repository code."
---

## Required `<project-root>/docs` reads

Shell-read before any database query, schema inspection, or read/write operation (reuse session reads): `<project-root>/docs/database-tools.md`. If a spec is missing, report the gap and use a documented fallback.

## Source of Truth

Treat the live database as source of truth — schema files, migrations, seeders, factories, and operations describe intent; the database records reality. Inspect live state before relying on code artifacts for data shape, relationships, constraints, or sample values.

Use the schema, query, and application-runtime operations configured in `database-tools.md`. Prefer read-only operations for inspection.

## Tool Inventory

The spec defines available connections, schema inspection, read-only queries, and application-runtime operations. Runtime operations may trigger application events or external services.

For failures depending on test data, inspect the test database rather than assuming local development data matches. Confirm the connection before querying.

## Discovery Workflow

1. Confirm the relevant connection.
2. Inspect only the tables, columns, indexes, views, or routines needed.
3. Query relevant counts, samples, or execution plans.

## Querying Guidelines

- `EXPLAIN` before trusting query cost — confirm index usage, especially on large tables.
- Qualify joined columns with table aliases.
- Apply the application's tenant, access, and soft-delete filters when relevant; raw queries do not inherit application scopes.
- `LIMIT` exploratory `SELECT`s — unbounded joins over large tables flood the response.
- Report `EXPLAIN` as prose ("Uses index X, scans N rows"), never raw output.

## Reconciling Counts

Trace the application's query path and reproduce its joins, filters, and scopes in the raw count. Investigate differences against evidence; do not assume either count is correct.

## Runtime Safety

- A user-approved workflow can authorize a bounded class of reversible writes to a verified local target. Reuse that authorization for each covered action; do not ask again just because a write uses a different command or tool. Outside that scope, obtain explicit consent before writing. Verify the effective target and expected effects. Irreversible destructive actions and real-world side effects require specific approval. Honor tool-enforced permissions.
- A rolled-back transaction does not undo external effects. Check events, jobs, mail, webhooks, and integrations before experiments.
- Batch independent runtime queries when initialization is expensive.
