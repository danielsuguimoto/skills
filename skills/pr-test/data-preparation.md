# Data preparation

Conditional reference for `SKILL.md` Step 6 — load only when its pointers fire.

## Already-applied artifacts

For an already-applied artifact, reconstruct a representative pre-state in the verified existing database. Reuse affected records still in their original state; apply [Step 6's fixture rule](SKILL.md#6-prepare-minimal-local-data) if none are suitable. If schema or execution tracking prevents a rerun, use targeted local commands or a project-supported rollback only if it preserves unrelated data and meets the safety boundary. If neither can reproduce the pre-state safely, follow the database reuse gate for the affected scenarios. Inspect the pre-state, have the selected driver run the artifact through its real entry point, and verify its effects under Step 7. A no-op against already-repaired data is not proof.

## Production copies

Copy production records into the verified existing local database only when a purpose scenario needs historical state that lightweight local preparation cannot reproduce. Follow `database-tools.md`: confirm source and destination; copy only needed records and relationships; preserve reproduction-critical values while masking unrelated sensitive data; isolate mail, queues, webhooks, and external integrations before use. Keep secrets out of evidence and version control; write only to local.
