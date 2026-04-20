# Blob Access Tiers

Azure Blob Storage provides multiple access tiers to optimize cost based on data access frequency.

## Tier Comparison

| Tier | Access Frequency | Min Duration | Read Cost | Storage Cost | Use Case |
|------|-----------------|-------------|-----------|-------------|----------|
| **Hot** | Frequent | None | Lowest | Highest (~$0.018/GB) | Active data, serving, processing |
| **Cool** | Infrequent (30+ days) | 30 days | Medium | Medium (~$0.01/GB) | Short-term backup, staging |
| **Cold** | Rare (90+ days) | 90 days | Higher | Lower (~$0.0045/GB) | Compliance, infrequent access |
| **Archive** | Almost never (180+ days) | 180 days | Highest (rehydrate needed) | Lowest (~$0.002/GB) | Long-term retention, regulatory |

## Early Delete Penalties

Moving a blob OUT of a tier before minimum retention charges remaining days:
- Cool tier blob deleted after 10 days → pay for 20 more days of Cool storage
- Cold tier blob deleted after 30 days → pay for 60 more days of Cold storage
- Archive blob deleted after 90 days → pay for 90 more days of Archive storage

**Key insight**: Don't set lifecycle policies to move blobs to Cool if they might be accessed within 30 days. Transaction cost + early delete penalty can exceed Hot storage costs.

## Rehydration (Archive Tier)

Archive tier requires rehydration before read — data is offline.

| Priority | Time | Cost | Use Case |
|----------|------|------|----------|
| **Standard** | Up to 15 hours | Lower | Planned access, batch processing |
| **High** | Under 1 hour | Higher (~10x) | Urgent access, incident response |

- Rehydrate to Hot or Cool (choose target tier)
- Copy-based rehydration: creates a copy in online tier (original stays in Archive)
- In-place rehydration: changes blob tier directly
- Plan ahead for archive access — build rehydration SLA into processes

## Smart Tiering (Access Time Tracking)

**Last access time tracking** enables intelligent tier management:
- Enable at the storage account level
- Tracks when each blob was last read
- Use with lifecycle management policies to auto-tier based on actual access patterns
- Small monitoring overhead per blob

**Lifecycle management auto-tiering**:
- Move to Cool after 30 days of no access
- Move to Cold after 90 days of no access
- Move to Archive after 180 days of no access
- Auto-promotes back on access (online tiers only)

## Break-Even Calculations

```
When does Cool save money over Hot?
─ Storage savings must exceed: (access cost increase × operations) + early delete risk
─ Rule of thumb: accessed less than once per month → Cool
─ Accessed less than once per quarter → Cold
─ Accessed less than twice per year → Archive

Small file warning:
─ Transaction costs dominate for small files
─ Moving 1M × 1KB files to Cool costs more in transactions than Hot storage savings
─ Rule: Don't tier files < 128KB unless truly dormant
```

## Tier Setting Methods

- **SetBlobTier API**: Change tier on individual blobs programmatically
- **Lifecycle management policies**: Rule-based automatic tiering (runs daily)
- **Upload with tier**: Set tier at upload time via x-ms-access-tier header
- **Batch operations**: Tier multiple blobs in a single API call
- **AzCopy**: Supports setting tier during copy operations

## Best Practices

1. **Default to Hot** for new data — optimize later with data
2. **Enable last access time tracking** on all production accounts
3. **Move to Cool after 30 days** of no access
4. **Move to Archive after 180 days** for retention data
5. **Set lifecycle management policies** from day 1
6. **Don't tier small files** (< 128KB) — transaction costs exceed savings
7. **Document rehydration SLA** for archived data before archiving
8. **Use prefix filters** in lifecycle rules to target specific data sets
9. **Monitor early delete penalties** in cost analysis
10. **Consider Cold tier** as middle ground between Cool and Archive

## Related References

- [lifecycle-management.md](lifecycle-management.md) — Automated tiering policies
- [storage-selection.md](storage-selection.md) — Service selection guide
- [performance.md](performance.md) — Tier impact on performance
