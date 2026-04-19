## Storage Service Selection Decision Tree

```
What kind of data?
├─ Unstructured objects (images, documents, backups, logs)?
│  ├─ Big data / analytics workload? → ADLS Gen2 (Blob + hierarchical namespace)
│  ├─ Standard object storage? → Blob Storage (GPv2)
│  └─ Archive / compliance retention? → Blob Storage with Archive tier + immutability
├─ File shares (SMB/NFS)?
│  ├─ Lift-and-shift from on-prem NAS? → Azure Files (standard or premium)
│  ├─ Enterprise NAS, SAP, HPC, sub-ms latency? → Azure NetApp Files
│  ├─ Linux NFS + low latency + ONTAP features? → Azure NetApp Files
│  └─ Simple shared config files / small shares? → Azure Files standard
├─ Disk storage (block)?
│  └─ VM-attached → Managed Disks (choose tier: Ultra, Premium SSD v2, Premium SSD, Standard SSD, Standard HDD)
└─ Queue / Table (legacy)?
   └─ Consider Service Bus (queues) or Cosmos DB (tables) instead — storage queues/tables are cheap but limited
```
