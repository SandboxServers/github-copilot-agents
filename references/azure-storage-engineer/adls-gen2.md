## ADLS Gen2 (Data Lake Storage)

### What It Actually Is
- NOT a separate service — it's Blob Storage with hierarchical namespace (HNS) enabled
- HNS enables: directory operations, atomic renames, POSIX ACLs, true directory structure
- Same pricing as Blob Storage + small HNS transaction premium
- Compatible with: Hadoop (ABFS driver), Spark, Databricks, Synapse, Data Factory, HDInsight

### When to Use ADLS Gen2 vs Plain Blob
| Use ADLS Gen2 When | Use Plain Blob When |
|--------------------|---------------------|
| Big data analytics (Spark, Databricks, Synapse) | Object storage (images, documents, backups) |
| Need directory-level ACLs | Simple RBAC is sufficient |
| Need atomic directory operations | No directory semantics needed |
| Data lake / medallion architecture | Application blob storage |
| Hadoop ecosystem integration | REST API / SDK access patterns |

### ADLS Gen2 Limitations
- Blob versioning NOT supported with HNS
- Blob snapshots NOT supported with HNS (only ADLS snapshots preview)
- Point-in-time restore NOT available with HNS
- Some Blob Storage features have limited support with HNS
- Cannot disable HNS once enabled
