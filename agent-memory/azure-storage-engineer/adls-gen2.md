# Azure Data Lake Storage Gen2

Blob Storage with hierarchical namespace (HNS) for big data analytics.

## What It Is

- Blob Storage with hierarchical namespace (HNS) enabled
- NOT a separate service — same underlying infrastructure
- Enables directory-level operations (rename, delete directory atomically)
- ACL-based access control (POSIX-like)
- Optimized for analytics workloads
- Same pricing as Blob + small HNS transaction premium

## Key Features

| Feature | Description |
|---------|-------------|
| **Hierarchical namespace** | True directories, not flat blobs with `/` delimiter |
| **POSIX-like ACLs** | Read/write/execute at file and directory level |
| **Atomic directory operations** | Rename/delete entire directories atomically |
| **Dual API compatibility** | Works with both Blob APIs and ADLS APIs |
| **ABFS driver** | `abfs://container@account.dfs.core.windows.net/path` |

## When to Use ADLS Gen2 vs Plain Blob

| Use ADLS Gen2 | Use Plain Blob |
|---------------|---------------|
| Big data analytics (Spark, Databricks, Synapse) | Object storage (images, docs, backups) |
| Need directory-level ACLs | Simple RBAC is sufficient |
| Need atomic directory operations | No directory semantics needed |
| Data lake / medallion architecture | Application blob storage |
| Hadoop ecosystem integration | REST API / SDK access patterns |

## Access Control Model

**RBAC** (coarse-grained) + **ACLs** (fine-grained):
- RBAC: Account or container level, for service-to-service access
- ACLs: Directory and file level, for user-level granularity
- Access ACL: Controls access to the item
- Default ACL: Inherited by new child items in a directory
- Use RBAC for service principals, ACLs for multi-team governance

## Integration Points

| Service | Integration Method |
|---------|-------------------|
| **Synapse Analytics** | Native ADLS Gen2 connector |
| **Databricks** | ABFS driver, direct mount |
| **HDInsight** | ABFS driver, Hadoop compatible |
| **Data Factory** | Copy activity, data flows |
| **Power BI** | DirectQuery or import |

## Medallion Architecture

```
Bronze (Raw)        → Silver (Cleansed)      → Gold (Curated)
─────────────────     ─────────────────────     ─────────────────
Raw ingestion         Validated, deduplicated   Business-level aggregates
Source format         Standardized schema       Star schema / metrics
Append-only           Type 2 SCD patterns       Read-optimized
Lowest ACL access     Processing team ACLs      Analyst/BI ACLs
Archive after proc    Cool after 90 days        Hot tier
```

- Separate containers or directories per layer
- ACLs per layer for access governance
- Lifecycle management per layer (archive bronze after processing)

## Limitations

- Blob versioning NOT supported with HNS
- Blob snapshots NOT supported with HNS
- Point-in-time restore NOT available with HNS
- Cannot disable HNS once enabled on an account
- Some Blob features have limited HNS support

## Best Practices

1. **Enable HNS at creation** — cannot convert existing account
2. **Plan directory structure** before loading data
3. **Use service principal or managed identity** for service access
4. **ACLs for multi-team** data lake governance
5. **Separate containers per medallion layer** for isolation
6. **Default ACLs on directories** for consistent inheritance
7. **Lifecycle management per layer** to optimize costs
8. **Monitor directory operation costs** (HNS transaction premium)

## Related References

- [storage-selection.md](storage-selection.md) — When to choose ADLS Gen2
- [security.md](security.md) — RBAC and access control details
- [lifecycle-management.md](lifecycle-management.md) — Tiering per layer
