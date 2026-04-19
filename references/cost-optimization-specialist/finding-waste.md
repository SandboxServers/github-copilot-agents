## Finding Waste — The Cost Audit Playbook

### Common Waste Patterns

| Waste Pattern | How to Find It | Fix |
|---|---|---|
| **Idle/underutilized VMs** | Advisor: CPU <5% for 7+ days | Right-size or deallocate |
| **Unattached disks** | Advisor: disks not attached to any VM | Delete after confirming no data needed |
| **Snapshots in Premium storage** | Advisor: snapshots stored on Premium | Migrate to Standard (60% savings) |
| **Oversized App Service Plans** | Cost Analysis: compare plan cost vs actual usage | Scale down or use autoscale |
| **Forgotten dev/test resources** | Tag-based filter: env=dev with high spend | Shut down, auto-stop schedules |
| **Orphaned public IPs** | Resource Graph: public IPs not associated to NIC | Delete |
| **Over-provisioned Cosmos DB RUs** | Advisor: RU recommendations, actual vs provisioned | Autoscale or reduce provisioned RUs |
| **ExpressRoute circuits not in use** | Resource Graph: circuits with provider status != Provisioned | Delete |
| **Log Analytics over-ingestion** | Monitor: data ingestion volume vs commitment tier | Reduce noisy sources, adopt commitment tiers |
| **Dev subscription costing more than prod** | Cost Analysis: compare subscription spend side-by-side | Investigate — this is always a red flag |

### The Cost Optimization Workbook
- Azure Advisor's **Cost Optimization Workbook** — centralized hub for cost recommendations
- Covers: VMs, AKS, App Service, Log Analytics, Synapse, disks, snapshots
- Identifies: idle resources, improperly deallocated VMs, orphaned resources
- Use as starting point for monthly cost reviews
