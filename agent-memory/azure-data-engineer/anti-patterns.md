# Anti-Patterns

## Ingestion Anti-Patterns

### Landing Raw Data Without Schema Validation
Ingesting data with no schema checks or data quality gates at the bronze layer. Bad data propagates downstream, corrupting silver and gold tables. By the time consumers notice, the root cause is buried under days of pipeline runs.
- **Why it happens:** Teams prioritize "get data in first, clean later." Bronze is treated as a dumping ground.
- **Consequence:** Downstream dashboards show wrong numbers. Root-cause analysis requires replaying days of pipeline history.
- **Fix:** Validate schema (column names, types, nullability) at ingestion. Use Delta schema enforcement or Fabric Dataflow Gen2 schema validation. Reject or quarantine malformed records immediately.
- **Reference:** Fabric materialzied lake views support built-in data quality constraints between layers.

### Not Using Incremental Loads
Running full-refresh pipelines for every execution — reloading millions of rows when only thousands changed. This wastes compute, inflates costs, and creates unnecessarily long pipeline durations.
- **Why it happens:** Full refresh is simpler to implement and reason about. Incremental requires tracking state.
- **Consequence:** A 100M-row table refresh takes hours instead of minutes. Fabric capacity units burn on redundant I/O.
- **Fix:** Use watermark columns, change data capture (CDC), or Fabric Copy Jobs for incremental extraction. Use `MERGE INTO` for Delta upserts instead of drop-and-reload.
- **Pattern:** `SELECT * FROM source WHERE modified_date > @last_watermark` — store watermark in pipeline metadata.

### Copy Activity Without Proper Parallelism
Using default parallelism in Data Factory Copy Activity. For large datasets, this results in single-threaded, slow transfers. The Fabric Warehouse connector limits concurrent queries to 32 — setting parallelism too high causes throttling.
- **Why it happens:** Default settings "just work" for small datasets. Nobody tunes until production volumes hit.
- **Fix:** Set Degree of Copy Parallelism to (DIU or IR nodes) × 2–4. Partition source data for parallel reads.
- **Warning:** Fabric Warehouse hard-limits concurrent queries to 32. Exceeding this causes `DWCopyCommandOperationFailed`.

### Not Handling Late-Arriving Data in Streaming
Streaming pipelines that assume data arrives in order. Late-arriving events get dropped or processed incorrectly, leading to inaccurate aggregates and missed SLAs.
- **Why it happens:** Event ordering is assumed, not verified. Watermark and lateness windows feel like over-engineering.
- **Consequence:** Real-time dashboards show incorrect counts. Alerts fire on phantom anomalies.
- **Fix:** Use watermarks and allowed-lateness windows in Eventstream or Spark Structured Streaming. Design idempotent writes so reprocessing late arrivals is safe.

## Architecture Anti-Patterns

### No Medallion Architecture
Dumping everything into a single layer — raw, transformed, and business-ready data coexist in the same tables or lakehouse. No clear lineage, no separation of concerns, no ability to rebuild downstream from raw.
- **Why it happens:** "We'll organize later" turns into never. Small teams start with one lakehouse and it grows organically.
- **Consequence:** Cannot rebuild gold tables when transformation logic changes. No isolation between raw ingestion and consumer-facing data.
- **Fix:** Implement bronze (raw), silver (cleansed), gold (business-ready) layers. Use separate lakehouses or schemas per layer. Fabric standardizes on Delta Lake format across all layers.
- **Fabric pattern:** Separate workspaces per layer. Use deployment pipelines for promotion.

### Over-Partitioning or Under-Partitioning Delta Tables
Over-partitioning creates thousands of tiny files (small file problem) that degrade read performance. Under-partitioning produces huge files that can't be pruned efficiently. Both waste compute on full scans.
- **Common mistake:** Partitioning by a column with millions of unique values (e.g., `user_id`) creates a folder per value.
- **Fix:** Partition by high-cardinality columns used in filters (e.g., date). Target ~1 GB per file after compaction.
- **Tooling:** Use `OPTIMIZE` and Z-ordering for further pruning. Enable predictive optimization for frequently queried managed tables.

### Manual Schema Evolution Instead of Delta Auto-Merge
Manually altering table schemas when source systems add columns. This creates brittle pipelines that break on any upstream change and requires coordinated deployments.
- **Why it happens:** Teams fear automatic schema changes will break consumers.
- **Fix:** Use Delta Lake `mergeSchema` or `autoMerge` options. Schema evolution handles new columns automatically.
- **Guard rail:** Validate breaking changes (type changes, dropped columns) separately with schema comparison checks.

### Mixing Production and Development in the Same Workspace
Dev pipelines writing to prod tables, test data polluting production lakehouses, or shared Spark clusters where a dev notebook starves a production job.
- **Why it happens:** Starting small, one workspace for everything. No governance until something breaks.
- **Consequence:** A dev `DELETE FROM` hits the production table. A runaway Spark job consumes all capacity.
- **Fix:** Separate Fabric workspaces per environment. Use deployment pipelines for promotion. Git-back workspaces for version control. Isolate capacity where budget allows.

## Compute Anti-Patterns

### Spark Clusters Always-On When Serverless Would Work
Running dedicated Spark pools or Databricks clusters 24/7 for workloads that run a few hours per day. Idle clusters burn money with zero utilization.
- **Cost reality:** A 4-node Standard_DS3_v2 cluster running 24/7 costs ~$1,200/month. The same workload on serverless may cost $50–100/month.
- **Fix:** Use Fabric Spark serverless compute (auto-scales, auto-terminates). For Databricks, use job clusters with auto-termination. Reserve interactive clusters for active development only.

### Using Data Factory for Complex Transformations
Attempting complex joins, pivots, windowed aggregations, or ML feature engineering inside Data Factory expression language. ADF expressions aren't a programming language — no loops, limited error handling, unreadable at scale.
- **Symptom:** Pipeline JSON with hundreds of nested `@if()` and `@concat()` expressions that no one can debug.
- **Fix:** Use Data Factory for orchestration and simple data movement. Push complex transformations to Spark notebooks, Dataflow Gen2, or SQL stored procedures.

## Governance Anti-Patterns

### Ignoring Data Lineage and Governance From the Start
Treating governance as a future problem. By the time hundreds of tables exist with no catalog entries, no lineage, and no ownership, retrofitting is extremely expensive.
- **Why it happens:** Governance feels like overhead when you have 10 tables. At 500 tables with 50 pipelines, it's a crisis.
- **Fix:** Enable Microsoft Purview integration from day 1. Tag tables with owners, descriptions, and sensitivity labels. Track lineage automatically through pipeline metadata.

### Not Implementing Data Retention Policies
Never deleting old data. Storage grows unbounded, queries slow down scanning years of irrelevant history, and compliance requirements (GDPR, data residency) are violated.
- **Why it happens:** "Storage is cheap" — until you're scanning 10 TB per query in Synapse serverless at $5/TB.
- **Fix:** Define retention policies per table at design time. Use Delta `VACUUM` with appropriate retention periods. Archive cold data to cheaper storage tiers. Automate retention enforcement in pipeline schedules.

## Pipeline Anti-Patterns

### Monolithic Pipelines With No Error Isolation
A single pipeline that ingests 50 sources, transforms everything, and loads all targets. One failure in source #23 kills the entire run and blocks all 49 other sources.
- **Why it happens:** Easier to build one pipeline than manage 50. Monitoring is simpler with one execution ID.
- **Consequence:** SLA breaches for unrelated data domains because of one flaky source.
- **Fix:** Use modular pipeline design — one pipeline per source or logical domain. Use a parent/orchestrator pipeline with `Execute Pipeline` activities. Set `continueOnError` for independent branches.

### No Idempotent Writes
Pipelines that append data on every run without checking for duplicates. Re-running a failed pipeline doubles the data for the successful portion.
- **Why it happens:** Append-only is the simplest write pattern. Deduplication adds complexity.
- **Consequence:** Double-counted revenue, inflated metrics, data quality incidents.
- **Fix:** Use Delta `MERGE` for upserts. Include a deduplication step keyed on a business key + ingestion timestamp. Design every pipeline to be safely re-runnable.

### Hardcoded Connection Strings and Secrets
Embedding passwords, storage account keys, or connection strings directly in pipeline parameters or notebook code.
- **Why it happens:** Works in dev. Nobody enforces rotation policy until a credential leak.
- **Fix:** Use Azure Key Vault for all secrets. Reference Key Vault linked services in ADF. Use Managed Identity where supported. Never store secrets in pipeline JSON or notebook variables.
