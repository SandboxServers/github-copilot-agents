# Azure Storage Engineer — Knowledge Index

> **Last Updated:** 2026-04-19 — Added gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Storage Selection | [storage-selection.md](storage-selection.md) | When choosing between Blob, ADLS Gen2, Files, NetApp Files, or Queue storage |
| Blob Access Tiers | [blob-access-tiers.md](blob-access-tiers.md) | When designing hot/cool/cold/archive tier strategies or Smart Tiering |
| Lifecycle Management | [lifecycle-management.md](lifecycle-management.md) | When configuring automatic tier transitions or data deletion policies |
| Redundancy | [redundancy.md](redundancy.md) | When choosing LRS/ZRS/GRS/GZRS or designing for regional failures |
| Performance | [performance.md](performance.md) | When troubleshooting storage throughput, IOPS, or latency |
| Security | [security.md](security.md) | When designing storage auth, private endpoints, CMK, immutable storage, or SAS |
| ADLS Gen2 | [adls-gen2.md](adls-gen2.md) | When working with hierarchical namespace, data lake patterns, or ACLs |
| Azure Files & NetApp Files | [azure-files-netapp.md](azure-files-netapp.md) | When choosing file share solutions — SMB/NFS, performance tiers, sync |
| Data Protection | [data-protection.md](data-protection.md) | When configuring soft delete, versioning, point-in-time restore, or snapshots |
| NetApp Files Patterns | [netapp-files-patterns.md](netapp-files-patterns.md) | When designing Azure NetApp Files deployments — tiers, replication, snapshots |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing storage designs for common mistakes |
| Gotchas | [gotchas.md](gotchas.md) | When encountering unexpected behaviors or hidden constraints |
| Best Practices | [best-practices.md](best-practices.md) | When designing production storage — security, cost, performance, resilience |
