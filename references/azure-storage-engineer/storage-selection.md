# Storage Selection Guide

Quick-reference for choosing the right Azure storage service based on workload requirements.

## Service Comparison Matrix

| Storage Service | Best For | Max Size | Performance | Cost |
|----------------|----------|----------|-------------|------|
| **Blob Storage** | Unstructured data, media, backups, archives | 5 PB per account | 20K RPS per account | Hot: ~$0.018/GB, Cool: ~$0.01/GB |
| **ADLS Gen2** | Big data analytics, hierarchical namespace | 5 PB | Optimized for analytics workloads | Similar to Blob + small premium |
| **Azure Files** | SMB/NFS file shares, lift-and-shift | 100 TiB per share | Standard: IOPS varies, Premium: provisioned IOPS | Standard: ~$0.06/GB, Premium: ~$0.16/GB |
| **Azure NetApp Files** | Enterprise NAS, SAP HANA, HPC, Oracle | 500 TiB per volume | Ultra: 128 MiB/s per TiB | ~$0.12-0.36/GB/month |
| **Table Storage** | Key-value NoSQL, cheap, simple | 500 TiB per account | 20K transactions/sec per partition | ~$0.045/GB |
| **Queue Storage** | Simple message queuing | 500 TiB per account | 20K messages/sec | ~$0.045/GB |

## Decision Tree

```
What kind of data do you need to store?
│
├─ Need file shares (SMB/NFS)?
│  ├─ Enterprise NAS, SAP HANA, HPC, sub-ms latency? → Azure NetApp Files
│  ├─ Lift-and-shift from on-prem? → Azure Files
│  └─ Simple shared config / small shares? → Azure Files Standard
│
├─ Need analytics with directory hierarchy?
│  └─ ADLS Gen2 (Blob + HNS enabled)
│
├─ Need object storage?
│  ├─ Active data, serving, processing → Blob Hot tier
│  ├─ Backup, staging → Blob Cool tier
│  └─ Archive, compliance → Blob Archive tier
│
├─ Need cheap NoSQL key-value?
│  └─ Table Storage (or Cosmos DB for global distribution)
│
├─ Need simple message queue?
│  └─ Queue Storage (or Service Bus for advanced patterns)
│
└─ Block storage (VM disks)?
   └─ Managed Disks (Ultra, Premium SSD v2, Premium SSD, Standard SSD, Standard HDD)
```

## Account Type Guidance

| Account Type | Supported Services | Recommended For |
|-------------|-------------------|----------------|
| **General-purpose v2 (GPv2)** | Blob, Files, Table, Queue | Default choice for most workloads |
| **Premium Block Blob** | Block blobs only | High-transaction, low-latency workloads |
| **Premium File Shares** | Azure Files only | Latency-sensitive file share workloads |
| **Premium Page Blob** | Page blobs only | VM disks (legacy, prefer Managed Disks) |
| **ADLS Gen2 (GPv2 + HNS)** | Blob with HNS, Files, Table, Queue | Data lake analytics workloads |

## Key Selection Factors

- **Access patterns**: Frequency determines tier (Hot/Cool/Cold/Archive)
- **Latency requirements**: Sub-ms → NetApp Files or Premium Blob; seconds OK → Standard
- **Protocol needs**: SMB/NFS → Azure Files or NetApp Files; REST/SDK → Blob
- **Data volume**: Large analytics → ADLS Gen2; general objects → Blob
- **Compliance**: Immutable storage → Blob with WORM policies
- **Budget**: Table/Queue cheapest for structured; Blob Archive cheapest for retention

## Planning Checklist

1. Identify data type (objects, files, blocks, structured)
2. Estimate volume today and growth rate
3. Determine access frequency and latency needs
4. Identify protocol requirements (REST, SMB, NFS, ABFS)
5. Define redundancy and DR requirements
6. Assess compliance and security needs
7. Plan account topology (separate by workload/environment)
8. Set lifecycle management strategy from day 1

## Related References

- [blob-access-tiers.md](blob-access-tiers.md) — Tier selection and cost optimization
- [redundancy.md](redundancy.md) — Redundancy options and failover
- [adls-gen2.md](adls-gen2.md) — Data Lake Storage details
- [azure-files-netapp.md](azure-files-netapp.md) — File share comparison
