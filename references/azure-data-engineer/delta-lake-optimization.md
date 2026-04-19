# Delta Lake Optimization

## File Management

- **Target file size**: 128MB-1GB per file (depending on query patterns)
- **Small file problem**: Too many small files = slow listing, high transaction cost. OPTIMIZE command compacts.
- **Z-ORDER**: Co-locates related data in files. Use on columns frequently in WHERE clauses.
- **V-ORDER** (Fabric): Write-time optimization for faster reads in Power BI.
- **Auto-compact**: Automatically compacts small files after writes (Databricks / Fabric).
- **Predictive optimization** (Databricks): Automatically runs OPTIMIZE and VACUUM.

## Partitioning Strategy

```
Should you partition?
├─ Table < 1TB? → Probably not. Delta statistics + Z-ORDER often enough.
├─ Table > 1TB with common date filters? → Partition by date (year/month/day)
├─ Table > 1TB with common categorical filters? → Partition by low-cardinality column
├─ High cardinality column (user_id, etc.)? → NEVER partition. Use Z-ORDER instead.
└─ Streaming ingestion? → Partition by ingestion date for lifecycle management
```

## Query Performance Diagnosis

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Slow full-table scans | No partition pruning | Add Z-ORDER on filter columns, partition if >1TB |
| Slow joins | Data skew, small files | OPTIMIZE, handle skew with salting or broadcast |
| High storage costs | Over-retention, no compaction | VACUUM, lifecycle policies, compact files |
| Timeout on aggregations | Wrong compute size | Scale up cluster, enable Photon for SQL |
| Slow writes | Too many partitions | Reduce partition granularity, enable auto-compact |
| Metadata operations slow | Too many files/partitions | OPTIMIZE, reduce partition count |
