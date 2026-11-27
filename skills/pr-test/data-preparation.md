# Data preparation

Conditional reference for `SKILL.md` Step 6 — load only when its pointers fire.

## Already-applied artifacts

For an already-applied artifact, reconstruct a representative pre-state and make the artifact runnable through its real entry point. Reuse affected records still in their original state; apply [Step 6's fixture rule](SKILL.md#6-prepare-minimal-local-data) if none are suitable. If schema or execution tracking prevents a rerun, use a project-supported rollback, targeted local commands, or an isolated local database with the required earlier schema and execution state. The selected driver runs any shipped artifact involved. Provision, clone, or rebuild disposable local state when that is the practical path, preserving unrelated existing data under the safety boundary without requesting setup permission. Inspect the reconstructed pre-state, execute the artifact, and verify its effects under Step 7. An already-run artifact or already-repaired data alone is not a blocker, and a no-op against repaired data is not proof of the original change.

## Production copies

Production copies are an option only when a purpose scenario needs historical state that lightweight local preparation cannot reproduce. Follow `database-tools.md`: confirm production source and local destination; copy only needed records and relationships; preserve reproduction-critical values while masking unrelated sensitive data; isolate mail, queues, webhooks, and external integrations before use. Keep secrets out of evidence and version control; write only to local.
