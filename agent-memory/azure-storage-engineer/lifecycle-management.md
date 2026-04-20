# Lifecycle Management

Automate blob tiering, deletion, and snapshot cleanup based on rules.

## How It Works

- **Rules engine**: Automatically tier, delete, or snapshot blobs based on conditions
- **Execution**: Runs once per day (can take up to 24 hours to complete)
- **Scope**: Applies to block blobs at the account level
- **Filtering**: Target specific containers via prefix match or blob index tags
- **Targets**: Current versions, previous versions, snapshots

## Rule Conditions

| Condition | Description | Requires |
|-----------|-------------|----------|
| **Last modified time** | Days since blob was last written | Nothing (always available) |
| **Last access time** | Days since blob was last read | Access tracking enabled |
| **Creation time** | Days since blob was created | Nothing (always available) |
| **Blob index tags** | Filter by tag key-value pairs | Tags set on blobs |

## Rule Actions

| Action | Description | Applies To |
|--------|-------------|------------|
| **tierToCool** | Move to Cool tier | Base blob, versions |
| **tierToCold** | Move to Cold tier | Base blob, versions |
| **tierToArchive** | Move to Archive tier | Base blob, versions |
| **delete** | Delete blob/snapshot/version | Base blob, versions, snapshots |

## Example Policy

```json
{
  "rules": [{
    "name": "move-to-cool-after-30-days",
    "type": "Lifecycle",
    "definition": {
      "filters": {
        "blobTypes": ["blockBlob"],
        "prefixMatch": ["container1/"]
      },
      "actions": {
        "baseBlob": {
          "tierToCool": { "daysAfterModificationGreaterThan": 30 },
          "tierToArchive": { "daysAfterModificationGreaterThan": 180 },
          "delete": { "daysAfterModificationGreaterThan": 365 }
        },
        "snapshot": {
          "delete": { "daysAfterCreationGreaterThan": 90 }
        }
      }
    }
  }]
}
```

## Common Policy Patterns

**Standard tiering cascade**:
- Cool after 30 days → Cold after 90 days → Archive after 180 days → Delete after 365 days

**Version cleanup** (critical when versioning enabled):
- Tier versions to Cool after 30 days → Delete versions after 90 days

**Snapshot cleanup**:
- Delete snapshots older than 30 days

**Access-based tiering** (requires last access time tracking):
- Cool after 30 days no access → Archive after 180 days no access

## Lifecycle + Data Protection Interactions

| Feature | Lifecycle Impact | Cost Trap |
|---------|-----------------|----------|
| **Versioning** | Every write creates version → capacity grows | MUST add version cleanup rules |
| **Soft delete** | Deleted blobs consume capacity during retention | Factor into cost estimates |
| **Point-in-time restore** | Requires versioning + soft delete + change feed | Cascade cost effect |
| **Snapshots** | Accumulate indefinitely without cleanup | Add snapshot delete rules |
| **Immutable storage** | Cannot delete blobs under hold/retention | Immutability overrides lifecycle |

## Best Practices

1. **Enable last access time tracking** if using access-based rules
2. **Apply prefix filters** to target specific data sets
3. **Start with longer durations**, tighten over time with monitoring data
4. **Monitor cost impact** before and after policy changes
5. **Combine with data protection**: set soft delete retention before lifecycle delete
6. **Always add version cleanup rules** when versioning is enabled
7. **Don't tier small files** (< 128KB) — transaction costs exceed savings
8. **Test policies** in non-production first
9. **Document policies** for operational awareness
10. **Review policies quarterly** and adjust based on actual access patterns

## Related References

- [blob-access-tiers.md](blob-access-tiers.md) — Tier details and costs
- [data-protection.md](data-protection.md) — Soft delete, versioning, PITR
- [anti-patterns.md](anti-patterns.md) — Common lifecycle mistakes
