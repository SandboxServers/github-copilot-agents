## Redundancy Options

```
                    Single Region                    Multi-Region
                ┌──────────────────┐          ┌──────────────────────┐
Single Zone     │ LRS              │          │ GRS / RA-GRS         │
                │ 3 copies, 1 zone │          │ 6 copies (3+3)       │
                │ 11 nines durable │          │ 16 nines durability  │
                │ Cheapest          │          │ Async repl to paired │
                └──────────────────┘          └──────────────────────┘
Multi-Zone      ┌──────────────────┐          ┌──────────────────────┐
                │ ZRS              │          │ GZRS / RA-GZRS       │
                │ 3 copies, 3 zones│          │ 6 copies (3z + 3)    │
                │ 12 nines durable │          │ 16 nines durability  │
                │ Zone failure prot│          │ Zone + region failure │
                └──────────────────┘          └──────────────────────┘
```

### Redundancy Selection Guide

| Requirement | Recommendation |
|-------------|---------------|
| Dev/test, non-critical | LRS |
| Production, single region, zone resilience | ZRS |
| Production, disaster recovery needed | GRS (failover) or RA-GRS (read access to secondary) |
| Production, zone + regional resilience | GZRS or RA-GZRS |
| Smart Tier | Requires ZRS, GZRS, or RA-GZRS |
| Regulatory: data must stay in region | ZRS (no geo-replication) |
| Read-heavy, need secondary read endpoint | RA-GRS or RA-GZRS |

### Failover Behavior
- **GRS/GZRS**: Async replication → RPO typically < 15 minutes but NOT guaranteed
- **RA-GRS/RA-GZRS**: Secondary is read-only, available immediately (stale by RPO window)
- **Customer-managed failover**: You initiate; secondary becomes primary; DNS updates
- **After failover**: Account becomes LRS/ZRS → must reconfigure geo-redundancy
- **Object replication**: Alternative to GRS for selective replication to different accounts/regions
