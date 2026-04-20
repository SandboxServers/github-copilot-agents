# Azure Files and NetApp Files

Managed file share services for SMB and NFS workloads.

## Azure Files Overview

Managed SMB (2.1, 3.0, 3.1.1) and NFS (4.1) file shares.

### Standard Tiers

| Tier | Storage Cost | Transaction Cost | Use Case |
|------|-------------|-----------------|----------|
| **Transaction-optimized** | Highest | Lowest | Transaction-heavy workloads |
| **Hot** | Medium | Medium | Balanced general purpose |
| **Cool** | Lowest | Highest | Cost-optimized, infrequent access |

### Premium Tier

- SSD-backed, provisioned IOPS and throughput
- Performance scales with share size
- Single-digit millisecond latency
- ~$0.16/GB/month

## Azure File Sync

- Sync Azure Files to on-premises Windows Server
- **Cloud tiering**: Keep hot data local, cold data in cloud
- **Multi-site sync**: Sync across multiple office locations
- **Centralized backup**: Backup via Azure Backup on cloud share
- Use for hybrid cloud file sharing scenarios

## Azure NetApp Files

Enterprise-grade NAS powered by NetApp ONTAP.

### Service Levels

| Level | Throughput per TiB | Latency | Use Case |
|-------|--------------------|---------|----------|
| **Standard** | 16 MiB/s | Low | General purpose |
| **Premium** | 64 MiB/s | Sub-ms | Production workloads |
| **Ultra** | 128 MiB/s | Sub-ms | SAP HANA, HPC, Oracle |

### Features

- NFS 3/4.1 and SMB support
- Near-instant snapshots (ONTAP-based)
- Cross-region replication
- Volume backup
- Sub-millisecond latency

## Comparison

| Criteria | Azure Files Premium | NetApp Files |
|----------|--------------------|--------------|
| **Protocol** | SMB, NFS 4.1 | NFS 3/4.1, SMB |
| **Latency** | Single-digit ms | Sub-ms |
| **Max share/volume** | 100 TiB | 500 TiB |
| **Max IOPS** | 100,000 | 460,000 |
| **Identity** | Entra ID, AD DS | AD DS, LDAP |
| **Snapshots** | Share-level | Volume-level (ONTAP) |
| **Replication** | Azure File Sync, GRS | Cross-region replication |
| **Use case** | General file shares | SAP HANA, HPC, Oracle, databases |
| **Cost** | ~$0.16/GB | ~$0.12-0.36/GB |

## Decision Guide

```
Need file shares?
├─ General purpose, lift-and-shift → Azure Files Standard
├─ Low-latency file shares → Azure Files Premium
├─ Hybrid (on-prem + cloud) → Azure Files + File Sync
├─ Enterprise NAS (SAP, Oracle, HPC) → Azure NetApp Files
├─ Sub-ms latency required → Azure NetApp Files
└─ NFS v3 required → Azure NetApp Files
```

## Best Practices

1. **Azure Files for most** file share needs
2. **NetApp Files for enterprise** NAS requirements (SAP, Oracle, HPC)
3. **Use Azure File Sync** for hybrid cloud scenarios
4. **Premium tier** for latency-sensitive workloads
5. **Right-size shares**: Premium IOPS scales with provisioned size
6. **Enable snapshots** for point-in-time recovery
7. **Private endpoints** for both services in production
8. **Monitor IOPS** and throughput against provisioned limits

## Related References

- [storage-selection.md](storage-selection.md) — Service selection guide
- [security.md](security.md) — Network security and identity
- [performance.md](performance.md) — Performance optimization
