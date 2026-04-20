# Delta Lake Optimization

Delta Lake is the default table format for Fabric and Databricks lakehouses.

## OPTIMIZE (File Compaction)

- Compacts many small files into fewer, larger files for faster reads
- Run after large ingestion batches or streaming micro-batches
- Syntax: `OPTIMIZE table_name`
- Target file size: 128MB-1GB depending on query patterns
- Small files problem: too many files = slow listing, high metadata overhead, expensive transactions
- Auto-compact: automatically compacts after writes (enable in Databricks/Fabric)
- Predictive optimization (Databricks): automatically schedules OPTIMIZE and VACUUM

## Z-ORDER (Data Co-location)

- Colocates related data within files for faster filtered reads
- Syntax: `OPTIMIZE table_name ZORDER BY (column1, column2)`
- Choose columns frequently used in WHERE clauses and JOIN conditions
- Effective for 1-4 columns; more columns reduce effectiveness
- Rewrite required — Z-ORDER runs as part of OPTIMIZE
- Best for high-cardinality columns where partitioning would create too many files

## VACUUM (Old File Cleanup)

- Removes old file versions no longer referenced by the transaction log
- Syntax: `VACUUM table_name RETAIN 168 HOURS`
- Default retention: 7 days (168 hours) — don't go below this for time travel safety
- WARNING: files removed by VACUUM cannot be recovered
- Schedule VACUUM regularly to prevent unbounded storage growth
- Always run after OPTIMIZE to clean up pre-compaction files

## Partitioning Strategy

- Partition by low-cardinality columns: date, region, status — not user_id
- Tables under 1TB often don't need partitioning — Z-ORDER is sufficient
- Over-partitioning creates small file problems (each partition has its own files)
- Liquid clustering (Databricks): replaces traditional partitioning with automatic data layout
- Streaming tables: partition by ingestion date for lifecycle management
- Rule of thumb: each partition should contain at least 1GB of data

## File Sizing

- Target 128MB-1GB per file for optimal read performance
- OPTIMIZE achieves correct file sizing automatically
- Small files (< 10MB) cause expensive metadata operations and slow scans
- Large files (> 2GB) reduce parallelism and waste memory on partial reads
- Monitor file sizes: `DESCRIBE DETAIL table_name` shows file count and size

## Statistics Collection

- `ANALYZE TABLE table_name COMPUTE STATISTICS` for query optimization
- Column-level statistics enable predicate pushdown and data skipping
- Delta automatically collects min/max stats on first 32 columns
- Reorder columns to put frequently filtered columns first
- Statistics help the query optimizer skip irrelevant files

## V-Order (Fabric)

- Write-time optimization format for faster reads in Power BI DirectLake mode
- Applied automatically when writing to Fabric Lakehouse tables
- Columnar sorting and encoding for optimal Power BI scan performance
- No manual configuration needed — Fabric handles it transparently
- Complementary to Z-ORDER (V-Order for BI reads, Z-ORDER for Spark queries)

## Schema Evolution

- Add columns: `ALTER TABLE table_name ADD COLUMNS (col_name data_type)`
- Merge with schema evolution: `.option("mergeSchema", "true")` during writes
- Schema enforcement prevents writes with mismatched schemas (safety net)
- Column renaming and type widening supported with column mapping enabled
- Always test schema changes in non-production first

## Performance Diagnosis

- Slow full-table scans → add Z-ORDER on filter columns, partition if over 1TB
- Slow joins → OPTIMIZE both tables, handle data skew with salting or broadcast
- High storage costs → VACUUM regularly, apply lifecycle policies, compact files
- Timeout on aggregations → scale up compute, enable Photon for SQL workloads
- Slow writes → reduce partition granularity, enable auto-compact
- Metadata operations slow → too many files/partitions, run OPTIMIZE
