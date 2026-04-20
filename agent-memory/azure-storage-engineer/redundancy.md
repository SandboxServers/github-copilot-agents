# Storage Redundancy

Azure Storage replicates data to protect against failures. Choose redundancy based on availability and DR requirements.

## Redundancy Options

| Redundancy | Copies | Scope | Durability | Availability | Use Case |
|-----------|--------|-------|-----------|-------------|----------|
| **LRS** | 3 copies | Single datacenter | 11 nines | 99.9% | Dev/test, easily reproducible data |
| **ZRS** | 3 copies | 3 availability zones | 12 nines | 99.9% | Production requiring zone resilience |
| **GRS** | 6 copies | Primary + paired region | 16 nines | 99.9% (99.99% with RA-GRS) | DR across regions |
| **GZRS** | 6 copies | 3 zones + paired region | 16 nines | 99.9% (99.99% with RA-GZRS) | Highest availability |

## Architecture Diagram

```
                    Single Region                    Multi-Region
                ┌──────────────────┐          ┌──────────────────────┐
Single Zone     │ LRS              │          │ GRS / RA-GRS         │
                │ 3 copies, 1 zone │          │ 6 copies (3+3)       │
                │ Cheapest          │          │ Async repl to paired │
                └──────────────────┘          └──────────────────────┘
Multi-Zone      ┌──────────────────┐          ┌──────────────────────┐
                │ ZRS              │          │ GZRS / RA-GZRS       │
                │ 3 copies, 3 zones│          │ 6 copies (3z + 3)    │
                │ Zone failure prot│          │ Zone + region failure │
                └──────────────────┘          └──────────────────────┘
```

## RA-GRS / RA-GZRS (Read-Access)

- **Read-access** to secondary region endpoint
- Secondary is **eventually consistent** (RPO ~15 minutes)
- Secondary endpoint: `accountname-secondary.blob.core.windows.net`
- Use for read-heavy workloads with DR requirements
- Failover promotes secondary to primary
- Design for stale reads — do not use secondary for primary workload

## Failover Behavior

| Aspect | Details |
|--------|--------|
| **Trigger** | Microsoft-initiated (major disaster) or customer-initiated (any reason) |
| **RPO** | ~15 minutes (not guaranteed) |
| **Process** | Secondary becomes primary, DNS updates propagate |
| **After failover** | Account becomes LRS in the new primary region |
| **Recovery** | Must re-enable GRS/GZRS after failover |
| **Data loss** | Possible — writes after last sync point may be lost |

## Customer-Initiated Failover

1. Initiate via Azure Portal, CLI, or PowerShell
2. Secondary promoted to primary
3. DNS records updated (may take time to propagate)
4. Account becomes LRS/ZRS in new region
5. Reconfigure geo-redundancy after failover completes
6. Test process regularly — don't wait for a real disaster

## Object Replication (Alternative)

- Async replication between storage accounts
- Cross-region or same-region
- Policy-based: filter by prefix, container
- More granular control than GRS
- Use for: selective DR, data locality, analytics offload

## Selection Guide

| Requirement | Recommendation |
|-------------|---------------|
| Dev/test, non-critical | LRS |
| Production, single region | ZRS |
| Production with DR | GRS or RA-GRS |
| Production, highest availability | GZRS or RA-GZRS |
| Data must stay in region (regulatory) | ZRS |
| Read-heavy with secondary | RA-GRS or RA-GZRS |
| Selective cross-region replication | Object replication |

## Best Practices

1. **ZRS minimum** for all production workloads
2. **GZRS** for critical data requiring highest availability
3. **Test failover process** regularly (customer-initiated)
4. **Design for eventual consistency** on secondary reads
5. **Don't rely on read-secondary** for primary workloads
6. **Document RPO/RTO** expectations for each storage account
7. **Consider object replication** for granular cross-region needs
8. **Monitor replication lag** via Last Sync Time property

## Related References

- [data-protection.md](data-protection.md) — Soft delete, versioning, backups
- [security.md](security.md) — Network and access security
- [anti-patterns.md](anti-patterns.md) — GRS without testing failover
