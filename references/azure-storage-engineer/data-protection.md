# Data Protection

Protecting Azure Storage data from accidental deletion, corruption, and loss.

## Feature Matrix

| Feature | Description | Cost Impact | Recommended Retention |
|---------|-------------|-------------|----------------------|
| **Container soft delete** | Recover deleted containers | Capacity during retention | 7+ days |
| **Blob soft delete** | Recover deleted blobs | Capacity during retention | 14+ days |
| **Blob versioning** | Auto-save on every write | Version per write → capacity growth | Enable + lifecycle cleanup |
| **Point-in-time restore** | Restore blobs to prior state | Requires versioning + soft delete + change feed | 1-7 days |
| **Change feed** | Append-only log of blob changes | Small capacity + transaction cost | Required for PITR |
| **Snapshots** | Manual point-in-time capture | Billed for unique blocks only | Lifecycle cleanup |
| **Immutability** | WORM compliance | No extra charge | Per regulatory requirement |

## Soft Delete

**Container soft delete** and **blob soft delete** are separate settings:
- Container: Recoverable after container deletion (1-365 days)
- Blob: Recoverable after blob or version deletion (1-365 days)
- Enable both on all production accounts
- Deleted data still consumes capacity during retention

## Versioning

- Automatic version created on every overwrite
- Previous versions remain accessible by version ID
- Each version consumes storage (billed for unique blocks)
- **Critical**: Must pair with lifecycle management to clean old versions
- Without cleanup rules, versioning costs grow unbounded

## Point-in-Time Restore

- Restore block blobs to a specific previous state
- Requires: versioning + change feed + blob soft delete all enabled
- Restore at container or account level
- Maximum retention: 365 days
- Longer retention = more cost (all dependencies retain longer)

## Change Feed

- Append-only log of all blob changes (create, modify, delete)
- Stored in `$blobchangefeed` container (special system container)
- Use for: audit trails, triggering workflows, replication
- Required dependency for point-in-time restore
- Retained according to change feed retention setting

## Object Replication

- Async replication between storage accounts
- Cross-region or same-region
- Policy-based: filter by prefix, container
- Use for: DR, data locality, analytics offload
- Does not replicate: snapshots, versions, soft-deleted blobs

## Recommended Configurations

```
Minimum (all production accounts):
├─ Container soft delete: 7 days
├─ Blob soft delete: 7 days
└─ Resource lock on storage account

Standard (most production workloads):
├─ Container soft delete: 14 days
├─ Blob soft delete: 14 days
├─ Blob versioning: Enabled
├─ Lifecycle rule: Delete versions > 90 days
└─ Resource lock on storage account

Maximum (compliance / critical data):
├─ Container soft delete: 30 days
├─ Blob soft delete: 30 days
├─ Blob versioning: Enabled
├─ Point-in-time restore: 7 days
├─ Change feed: Enabled
├─ Immutability policies (per container/version)
├─ Lifecycle rule: Delete versions > 90 days
└─ Resource lock on storage account
```

## Best Practices

1. **Enable soft delete** (containers: 7+ days, blobs: 14+ days) on all production accounts
2. **Enable versioning** for critical data with lifecycle cleanup rules
3. **Enable point-in-time restore** for important containers
4. **Backup strategy**: versioning + soft delete + object replication to another region
5. **Resource locks** on all production storage accounts
6. **Monitor capacity impact** of versioning and soft delete retention
7. **Test restore procedures** before you need them
8. **Document recovery SLAs** for each data protection tier

## Related References

- [lifecycle-management.md](lifecycle-management.md) — Version and snapshot cleanup
- [redundancy.md](redundancy.md) — Replication-based protection
- [security.md](security.md) — Immutable storage details
