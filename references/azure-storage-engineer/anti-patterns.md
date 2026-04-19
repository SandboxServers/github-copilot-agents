## Common Mistakes and Anti-Patterns

1. **Enabling versioning without lifecycle rules** → Capacity explodes, costs skyrocket, no one notices for months
2. **Using storage account keys in application code** → Full access, no audit trail, impossible to rotate without downtime
3. **Public blob access enabled "temporarily"** → It's never temporary. Disable at account level.
4. **Same storage account for everything** → Hits IOPS/throughput limits, single blast radius, impossible to apply different policies
5. **Not planning blob naming** → Sequential names (timestamp prefix) create hot partitions; use hash prefix for high-throughput
6. **Archiving small files individually** → Transaction cost to archive + rehydrate exceeds storage savings; tar/zip first
7. **Ignoring soft delete retention in cost estimates** → Deleted data still costs money during retention period
8. **LRS for production workloads** → Single-zone failure loses data; ZRS minimum for production
9. **Not using private endpoints** → Data traverses public internet even within Azure
10. **Over-replicating** → GRS when data can be regenerated from source; pay 2x storage for data you don't need to protect

## Questions This Specialist Always Asks

### Scoping
- What type of data? (Objects, files, blocks, structured?)
- How much data today? Growth rate? (Affects account planning and cost projections)
- Access patterns? (Frequency, read/write ratio, hot vs cold split)
- Latency requirements? (Sub-ms → NetApp Files or Premium Blob; seconds okay → Standard Blob)

### Security
- Who needs access? (Users, applications, external parties, CI/CD?)
- How is access authenticated today? (Keys, SAS, managed identity, anonymous?)
- Compliance requirements? (Immutability, CMK, data residency, regulatory retention)
- Network access? (Public internet, VNet only, on-prem hybrid?)

### Cost
- Current monthly storage spend? (Baseline for optimization)
- Are lifecycle policies in place? (If not, there's almost certainly waste)
- Versioning enabled? With cleanup rules? (If yes without, that's the cost problem)
- How many storage accounts? (Too few → limits; too many → management overhead)

### Resilience
- RPO / RTO requirements? (Drives redundancy choice)
- Can data be regenerated from source? (If yes, LRS may suffice)
- Multi-region requirements? (Drives GRS/GZRS vs ZRS decision)
- Backup strategy? (Azure Backup for Blobs, snapshots, or cross-account replication?)
