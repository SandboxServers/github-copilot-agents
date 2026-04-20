# Waste Elimination

Waste elimination removes resources that provide zero or minimal value relative to their cost.
It is the fastest path to savings — no architecture changes, no risk, just cleanup.

## Common Waste Patterns

| Waste Type | Detection Method | Typical Savings |
|---|---|---|
| **Unattached managed disks** | Azure Advisor, Resource Graph | 100% of disk cost |
| **Idle public IP addresses** | No associated NIC or load balancer | ~$3.65/month each (adds up at scale) |
| **Oversized VMs** | CPU < 5% average for 7+ days | 30–70% per VM after right-sizing |
| **Non-production running 24/7** | No auto-shutdown configured | 65%+ with 10-hour/day schedule |
| **Orphaned resources** | No traffic, no connections for 30+ days | 100% of resource cost |
| **Unused App Service plans** | No apps deployed on the plan | 100% of plan cost |
| **Soft-deleted Key Vaults** | Purge protection expired, no recovery needed | Small per-vault, adds up across environments |
| **Old snapshots** | Created > 30 days ago with no retention policy | Storage costs (Premium snapshots especially) |
| **Log Analytics over-ingestion** | Noisy diagnostic settings, verbose logging | 20–50% of monitoring cost |
| **Stopped VMs with Premium disks** | VM deallocated but disk still billed | Full disk cost continues |
| **Unused Application Gateways** | No backend pools or zero traffic | ~$250/month for WAF v2 |
| **Orphaned NICs and NSGs** | Not attached to any VM or subnet | Small cost but indicates cleanup debt |
| **ExpressRoute circuits not in use** | Provider status != Provisioned | Full circuit cost ($200–$10,000/month) |

## Automated Detection

### Azure Advisor Cost Recommendations
- Built-in detection for underutilized VMs, unattached disks, idle resources
- Recommendations refresh daily — review weekly as part of governance cadence
- Cost Optimization Workbook aggregates recommendations across subscriptions
- Estimated monthly savings shown per recommendation

### Azure Resource Graph Queries
- Query for unattached disks: `resources | where type == "microsoft.compute/disks" | where properties.diskState == "Unattached"`
- Query for orphaned public IPs: `resources | where type == "microsoft.network/publicipaddresses" | where properties.ipConfiguration == ""`
- Query for orphaned NICs: `resources | where type == "microsoft.network/networkinterfaces" | where isnull(properties.virtualMachine)`
- Schedule queries via Azure Automation for weekly orphaned resource reports

### Custom Azure Policy
- **Audit unattached disks** — policy flags disks not attached to any VM
- **Audit public IPs without association** — flags idle public IPs
- **Deny Premium storage in dev/test** — prevent waste at deployment time
- **Require auto-shutdown tag** — ensure non-production VMs have shutdown schedules

## Log Analytics Optimization

Log Analytics ingestion is one of the most commonly overlooked cost drivers.

### Detection
- Check data ingestion volume in Log Analytics workspace → Usage and Estimated Costs
- Identify top tables by ingestion volume — often AzureDiagnostics, ContainerLog, Heartbeat
- Compare ingestion against commitment tier — are you paying per-GB above a tier threshold?

### Reduction Strategies
- Disable verbose diagnostic settings on non-critical resources
- Use resource-specific diagnostic settings instead of legacy AzureDiagnostics table
- Filter noisy log categories at the diagnostic setting level
- Enable Application Insights adaptive sampling to reduce telemetry volume
- Set appropriate log levels in application code — don't log DEBUG in production
- Use Basic Logs tier for high-volume, low-query tables (90% cost reduction, limited query)

## Cleanup Process

### Weekly Review
- Review Azure Advisor cost recommendations — action or dismiss with justification
- Check for new unattached disks and orphaned resources via Resource Graph
- Verify auto-shutdown is running on all non-production VMs

### Monthly Sweep
- Run full orphaned resource report across all subscriptions
- Review resources with zero traffic or connections in the past 30 days
- Check for stopped VMs still incurring disk costs — delete disks or convert to Standard
- Audit snapshot retention — delete snapshots older than policy allows

### Quarterly Deep Review
- Review with application owners — validate all resources are still needed
- Challenge resource group purpose — are there entire RGs that can be decommissioned?
- Audit ExpressRoute and VPN Gateway utilization
- Review Log Analytics ingestion trends and commitment tier fit
- Update waste elimination policies based on new patterns discovered
