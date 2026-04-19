## Azure Files vs Azure NetApp Files

| Factor | Azure Files | Azure NetApp Files |
|--------|------------|-------------------|
| **Max IOPS** | 100,000 (premium) | 460,000 (regular volume) |
| **Max throughput** | 10 GiB/s (premium) | 4.5 GiB/s regular, 12.5 GiB/s large volume |
| **Latency** | Single-digit ms (premium), variable (standard) | Sub-ms |
| **Max share size** | 100 TiB | 100 TiB regular, 2 PiB large |
| **Protocols** | SMB 2.1/3.x, NFSv4.1, REST | SMB 2.1/3.x, NFSv3, NFSv4.1, dual-protocol, S3-compatible |
| **Snapshots** | Share-level snapshots | Volume-level snapshots (ONTAP-based, near-instant) |
| **Replication** | Sync (Azure File Sync), GRS | Cross-region replication, cross-zone replication |
| **Identity** | Entra ID, AD DS, Entra Domain Services | AD DS, LDAP |
| **Pricing model** | Per-GiB provisioned (premium) or used (standard) | Per-GiB provisioned (capacity pool) |
| **Best for** | General file shares, lift-and-shift, Azure File Sync | SAP HANA, HPC, databases, enterprise NAS, low-latency |
| **Managed by** | Azure Storage platform | NetApp ONTAP on bare metal in Azure DC |
