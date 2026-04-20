# Gotchas

## Fabric Platform Gotchas

### Fabric Capacity — Pausing Doesn't Stop Scheduled Refreshes
Pausing a Fabric capacity stops compute, but scheduled refreshes (semantic models, dataflows, pipelines) still attempt to run. They fail with cryptic errors, generate noise in monitoring, and can silently break downstream dependencies.
- **Impact:** Teams assume pausing capacity is a clean cost-saving measure. It isn't — you must also disable schedules or use deployment pipelines to manage environment lifecycle.
- **Workaround:** Before pausing capacity, disable all scheduled refreshes. Use Power Automate or the Fabric REST API to automate schedule toggling alongside capacity pause/resume.
- **Throttling risk:** If CU consumption exceeds the capacity limit, throttling delays or rejects transactions temporarily — even without a pause.

### Lakehouse vs Warehouse in Fabric — Different Engines, Different SQL
A Fabric Lakehouse uses Spark + an auto-generated SQL analytics endpoint (read-only T-SQL). A Fabric Warehouse uses a full T-SQL engine with DML support. They have different SQL capabilities, different performance profiles, and different security models.
- **Key difference:** Lakehouse SQL endpoint is read-only. You cannot `INSERT`, `UPDATE`, or `DELETE` through it. Warehouse supports full transactional DML.
- **Security model:** Lakehouse supports object-level, column-level, and row-level security through the SQL endpoint. Warehouse has native T-SQL security.
- **Decision:** If consumers need write-capable SQL, use Warehouse. If Spark-first processing with SQL read access is sufficient, use Lakehouse.

### OneLake Shortcuts — Read-Only, No Write-Back
Shortcuts in OneLake are references to data in other locations. They appear as folders but are read-only — you cannot write data back through a shortcut. Deleting a shortcut does not delete the target data, but moving or deleting the target breaks the shortcut silently.
- **Name restrictions:** Shortcuts don't support non-Latin characters. Delta tables with spaces in the name won't be recognized. Names cannot contain `%` or `+`.
- **Limits:** Max 100,000 shortcuts per item. Max 10 shortcuts per single OneLake path. Max 5 direct shortcut-to-shortcut chains.
- **Propagation:** A new shortcut syncs almost instantly, but propagation varies by source performance and network. Table API recognition may take up to a minute.

### Fabric Workspace Git Integration — 90-Day Token Expiry
Dataflow Gen2 staging artifacts lose access if the creating user hasn't logged into Fabric for 90+ days. This causes refresh failures with misleading permission errors (`"insufficient permissions for accessing staging artifacts"`).
- **Fix:** The specific user mentioned in the error must log back into Fabric. If they've left the org, open a support ticket.
- **Broader issue:** Downstream items consuming Dataflow Gen2 via the Dataflows connector use an internal API prone to intermittent timeouts. Error messages like `"The key didn't match any rows in the table"` are misleading. Route output to a Lakehouse/Warehouse destination instead.

### Fabric Data Factory — Missing ADF Features
Migrating from Azure Data Factory to Fabric Data Factory? Several features are not yet available:
- Tumbling window triggers — not supported. Use scheduled or event triggers with workarounds.
- Mapping Data Flow activity and SSIS integration runtime — not available in Fabric.
- Web activity doesn't support service principal auth. GetMetadata/Script can't source from KQL databases.
- OAuth and Azure Key Vault aren't supported in Fabric connectors. MSI only for Azure Blob Storage currently.

## Delta Lake Gotchas

### Vacuum Default Retention — 7 Days, Time Travel Breaks If Too Aggressive
`VACUUM` removes files older than the retention threshold (default 7 days). If you vacuum too aggressively (e.g., 0 hours), time travel breaks immediately — any query referencing historical versions fails with `FileNotFoundException`.
- **Safe practice:** Never set retention below 7 days in production. If downstream jobs read older versions, extend retention to match the longest read window.
- **Configuration:** `delta.deletedFileRetentionDuration` per table. `spark.databricks.delta.retentionDurationCheck.enabled = false` disables the safety check (dangerous).
- **Interaction:** `VACUUM` only removes data files. The transaction log (`_delta_log/`) is managed separately by checkpointing.

### Parquet vs Delta — Delta IS Parquet Plus Transaction Log
Delta Lake stores data as Parquet files plus a `_delta_log/` transaction log directory. It's not a different file format — it's Parquet with ACID transactions, schema enforcement, and time travel layered on top.
- **Implication:** Any tool that reads Parquet can read the raw data files, but only Delta-aware tools understand which files are current vs tombstoned.
- **Fabric default:** Every engine in Fabric writes Delta by default with V-Order optimization. You're already using Delta whether you realize it or not.

### Spark OOM Is Usually Partition Skew, Not Insufficient Memory
When Spark jobs fail with `OutOfMemoryError`, the instinct is to increase executor memory. In most cases, the real problem is partition skew — one partition holds vastly more data than others, overwhelming a single executor.
- **Diagnosis:** Check partition sizes with `spark.sql("DESCRIBE DETAIL table")`. Look for max file sizes 10x+ the median.
- **Fix:** Use `REPARTITION` or salting to distribute skewed keys. Run `OPTIMIZE` on Delta tables to rebalance file sizes.
- **Prevention:** Avoid partitioning by high-skew columns (e.g., `country` where 80% of data is one value).

### Delta Lake Column Name Case Sensitivity
Delta Lake does not support case-sensitive column names. Columns like `MyColumn` and `mycolumn` create duplicate column errors in Dataflow Gen2 even though Parquet allows them.
- **Fix:** Standardize all column names to lowercase before landing in Delta tables.

## Synapse & Query Gotchas

### Synapse Serverless — Billed Per TB Scanned, Format Matters
Synapse serverless SQL pools charge per TB of data scanned, not per query or per hour. Querying CSV files or uncompressed formats over large datasets generates massive bills. Queries without partition pruning scan everything.
- **Cost math:** 1 TB scanned = ~$5. A daily query scanning 500 GB of CSV = ~$75/month for one query.
- **Fix:** Store data as Delta or Parquet (columnar, compressed). Use partition elimination in WHERE clauses. Create views with `OPENROWSET` targeting specific partitions. Set cost management alerts.

### Unity Catalog vs Purview — Overlapping Governance, No Native Sync
Unity Catalog (Databricks) and Microsoft Purview (Fabric/Azure) both provide data governance, cataloging, and lineage. They overlap significantly but don't natively sync with each other.
- **Decision:** Databricks-only → Unity Catalog. Fabric-only → Purview. Hybrid → Purview as the enterprise catalog, but expect manual effort to bridge governance metadata across platforms.

### Data Factory Expression Language — Gotchas in Practice
- No `try/catch` — pipeline-level error handling only. A failed expression crashes the entire activity.
- Variables are pipeline-scoped, not activity-scoped. Race conditions with `Set Variable` in parallel ForEach.
- `@utcnow()` evaluates once per expression, not once per row. Timestamp logic across activities may differ.
- String concatenation of `null` values doesn't fail — it silently produces empty strings.

### Linked Service Connection Pooling Varies by Connector
Different ADF connectors handle connection pooling differently. Some maintain persistent connections, others create new connections per activity execution. High-frequency pipelines against connectors without pooling can exhaust source system connection limits.
- **Watch for:** SQL Server, Oracle, and REST connectors behave differently under concurrency.
- **Fix:** Test connection behavior under concurrent pipeline runs. Use Self-Hosted IR connection limits to throttle if needed.

## Capacity and Billing Gotchas

### Fabric Capacity Throttling Is Cumulative
Fabric capacity units (CUs) are shared across all workloads in a capacity — Spark, pipelines, SQL, Power BI. A heavy Spark job can throttle Power BI report rendering. Throttling is delayed (smoothed over time), so you may not see impact immediately.
- **Gotcha:** Throttling applies to the entire capacity, not individual workspaces. One runaway workload affects everything.
- **Fix:** Monitor CU usage in Fabric Capacity Metrics app. Separate critical workloads into dedicated capacities.

### OneLake Storage Billing — Shortcuts Don't Move the Bill
OneLake storage is billed to the capacity that physically stores the data. Shortcuts reference data across capacities, but storage billing stays with the source. Compute (transaction costs) for reading through shortcuts is billed to the consuming capacity.
- **Gotcha:** If the source capacity is paused, shortcuts from other active capacities can still read the data — but reading from the source capacity directly fails.

### Synapse Dedicated SQL Pool — Paused Still Costs for Storage
Pausing a dedicated SQL pool stops compute billing but storage costs continue. Distribution-replicated tables and materialized views still incur storage charges. Resuming requires warm-up time.

### Data Factory Self-Hosted IR — Single Point of Failure
A single Self-Hosted Integration Runtime node is a single point of failure. If the VM hosting it goes down, all pipelines using on-premises sources fail.
- **Fix:** Deploy SHIR in high-availability mode with multiple nodes. Monitor node health. Use auto-update to keep IR versions current.

## Spark Gotchas

### Spark Session Configuration Doesn't Persist Across Cells
In Fabric notebooks, `spark.conf.set()` applies to the current session only. Restarting the session or running a different notebook resets all configurations. Don't rely on session-level configs for production pipelines.
- **Fix:** Set configurations in the Spark pool/environment settings or at the top of every notebook with explicit `spark.conf.set()` calls.

### Notebook Output Limits Can Hide Errors
Fabric notebooks truncate large outputs in the UI. If a transformation produces warnings or partial errors, they may be invisible in the truncated display. The pipeline shows "success" while data quality silently degrades.
- **Fix:** Log critical metrics (row counts, error counts) explicitly to a Delta audit table. Don't rely on notebook cell output for production monitoring.
