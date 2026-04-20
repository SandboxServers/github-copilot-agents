# Best Practices

## Architecture

### Medallion Architecture as Default Pattern
Organize data into bronze (raw), silver (cleansed/enriched), gold (business-ready) layers. Each layer has clear ownership, quality expectations, and access patterns.
- **Bronze:** Preserve all source data exactly as received. Append-only, schema-on-read. Use Delta format even at bronze for auditability and time travel. Partition by ingestion date for efficient data management.
- **Silver:** Remove duplicates, enforce schema, standardize types and formats. Join across sources. Implement business validation rules. Use Unity Catalog managed tables.
- **Gold:** Pre-aggregated, denormalized, access-controlled. Tailored to specific consumers (BI, ML, APIs). Published as data products with clear ownership.
- **Fabric guidance:** Use separate lakehouses per layer within separate workspaces for governance isolation. Bronze can use shortcuts instead of copying data when the source is OneLake, ADLS Gen2, S3, or GCS.
- **Materialized lake views:** Fabric supports declarative pipelines between layers. Define SQL transformations; Fabric handles dependency ordering, incremental refresh, and quality rules automatically.

### Delta Lake as Default Storage Format
Use Delta Lake for all structured and semi-structured data. Delta provides ACID transactions, schema enforcement, schema evolution, time travel, and efficient upserts via `MERGE`.
- Fabric standardizes on Delta format — every engine writes Delta by default with V-Order optimization for fast reads across Power BI, SQL, Spark, and other engines.
- Avoid raw Parquet for tables that need updates, deletes, or transactional guarantees.
- Use Unity Catalog volumes or OneLake Files section for unstructured landing zones.
- Delta is Parquet + transaction log. Any Parquet reader can read the data files, but you need Delta-aware tools for transactional semantics.

### Incremental Processing Patterns
Never full-refresh what you can process incrementally. Common patterns:
- **Watermark columns:** Track `last_modified_date` or `row_version` to extract only changed records. Store watermarks in pipeline metadata.
- **Change Data Capture (CDC):** Use Fabric Mirroring or database CDC features to capture inserts/updates/deletes from source systems.
- **Delta MERGE:** `MERGE INTO target USING source ON key WHEN MATCHED THEN UPDATE WHEN NOT MATCHED THEN INSERT` — idempotent upserts that handle late-arriving data gracefully.
- **Fabric Copy Jobs:** Managed incremental data movement that replaces ADF CDC artifacts. Fabric handles change tracking internally.
- **Streaming:** Eventstream with checkpointing for continuous incremental processing. KQL update policies for real-time transformation in Eventhouse.

## Data Quality & Governance

### Validate at Ingestion, Not After Consumption
Data quality checks belong at the pipeline boundary — when data enters bronze or transitions to silver. Catching issues after consumers report them means days of bad data in production.
- Validate schema (expected columns, types, nullability) at bronze ingestion. Use Delta schema enforcement to reject malformed records.
- Apply business rules (range checks, referential integrity, uniqueness) at silver promotion.
- Implement quarantine tables for rejected records with error metadata for investigation.
- Use Fabric materialized lake views with built-in data quality constraints where applicable.
- Track quality metrics: null rates, duplicate counts, row count variance from expected ranges.

### Governance From Day 1 — Purview and Sensitivity Labels
Retrofit governance is 10x harder than building it in. Start with:
- **Microsoft Purview:** Register data sources, enable automated scanning, catalog all datasets. Use Purview for enterprise-wide discovery across Fabric, Azure, and multi-cloud.
- **Sensitivity labels:** Apply Microsoft Information Protection labels to classify data (public, internal, confidential, restricted). Labels flow through OneLake.
- **Data domains:** Use Fabric domains to organize assets by business area (sales, finance, ops). Assign domain owners responsible for quality and governance.
- **Lineage:** Enable automatic lineage tracking. Pipeline metadata captures source-to-target mappings. Unity Catalog tracks table-to-table dependencies automatically.
- **Access control:** Use workspace roles (Admin, Member, Contributor, Viewer) with least-privilege. Enable row-level and column-level security on gold layer tables.
- **Data catalog:** Add descriptions, tags, and owner metadata to all catalogs, schemas, and tables. Use OneLake Catalog for centralized discovery.

## Cost Optimization

### Use Serverless Where Possible
Dedicated compute running 24/7 for intermittent workloads is the single biggest cost waste in data platforms.
- **Fabric Spark:** Serverless by default — auto-scales, auto-terminates. No cluster management needed. Pay only for active compute.
- **Synapse serverless SQL:** Pay-per-TB-scanned for ad-hoc queries. Store data in Delta/Parquet to minimize scan volume. Avoid CSV/JSON for queryable data.
- **Databricks:** Use job clusters with auto-termination for scheduled workloads. Reserve interactive clusters for active development sessions only.
- **Fabric capacity:** Right-size SKUs using monitoring data. Use capacity pause for non-production workspaces — but disable all schedules first to avoid failed refresh noise.

### Optimize Delta Table Performance
- Run `OPTIMIZE` regularly to compact small files (target ~1 GB per file). Small file problem is the #1 Delta performance issue.
- Use Z-ordering on columns frequently used in filters and joins — enables data skipping for faster queries.
- Set `VACUUM` retention to match your time-travel and audit requirements (minimum 7 days, extend for compliance).
- Enable predictive optimization for frequently queried managed tables in Databricks.
- Use V-Order (automatic in Fabric) for cross-engine read performance optimization.

## Monitoring & Operations

### Pipeline Monitoring and Data Freshness SLAs
- Set alerts on pipeline failures — include timeout and partial-success states, not just errors.
- Define data freshness SLAs per gold table (e.g., "refreshed within 2 hours of source update"). Measure against actual refresh completion times.
- Track data quality metrics as pipeline outputs: null rates, duplicate counts, schema drift frequency, row count deltas.
- Use Fabric Monitoring Hub for cross-workspace pipeline visibility with drill-down into copy activity duration breakdowns.
- Set cost management alerts on Synapse serverless and Fabric capacity to catch runaway workloads before month-end surprises.

### Version Control — Git-Backed Workspaces
- Connect Fabric workspaces to Git repos for source control of pipelines, notebooks, and dataflows. Cherry-pick individual items for check-in — no all-or-nothing commits.
- Use Fabric deployment pipelines to promote across dev → test → prod without manual reconfiguration. No external Git repo required (optional).
- For ADF, export ARM templates and manage via CI/CD pipelines in Azure DevOps or GitHub Actions. Fabric CI/CD is simpler — no ARM template dependency.
- Document pipeline dependencies and data flow DAGs. Use lineage tracking to understand blast radius before making changes.

## Pipeline Design

### Modular, Idempotent Pipelines
- Design every pipeline to be safely re-runnable. Use Delta `MERGE` for upserts, not blind appends.
- One pipeline per source or logical domain. Orchestrate with a parent pipeline using `Execute Pipeline` or `Invoke Pipeline` activities.
- Set `continueOnError` for independent branches so one source failure doesn't block unrelated domains.
- Log pipeline metadata (start time, end time, rows processed, errors) to an audit table for operational visibility.

### Secrets and Connection Management
- Store all secrets (passwords, keys, connection strings) in Azure Key Vault. Reference Key Vault linked services.
- Use Managed Identity for authentication where supported — eliminates credential rotation overhead.
- Never hardcode secrets in pipeline JSON, notebook variables, or Dataflow parameters.
- Rotate service principal secrets on a schedule and test rotation in non-production first.

### Testing and Validation
- Test pipelines with representative data volumes before production — performance characteristics change dramatically at scale.
- Validate row counts and checksums between source and destination after each pipeline run.
- Use Fabric deployment pipelines to test configuration changes in staging before promoting to production.
- Implement circuit breakers: if a pipeline fails N times consecutively, alert and pause rather than retry indefinitely.
