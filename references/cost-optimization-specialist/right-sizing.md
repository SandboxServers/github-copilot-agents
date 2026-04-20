# Right-Sizing

Right-sizing is the highest-impact cost optimization activity. It reduces waste at the source
by matching resource capacity to actual workload demand. Always right-size before buying commitments.

## Azure Advisor Recommendations

Azure Advisor is the starting point for right-sizing across all services.

### How It Works
- Analyzes CPU, memory, network, and disk metrics over 7+ days
- Flags VMs with < 5% average CPU utilization as candidates for resize or shutdown
- Provides specific SKU recommendations with estimated monthly savings
- Recommendations refresh daily — review weekly as part of cost governance cadence

### Automation
- Export Advisor recommendations via REST API or Azure Resource Graph
- Use Azure Automation runbooks to auto-resize or deallocate flagged resources
- Integrate with Azure DevOps or GitHub Actions for approval workflows
- Set up Advisor alerts to notify when new cost recommendations appear

## VM Right-Sizing

VMs are typically the largest cost category and the biggest right-sizing opportunity.

### Assessment Process
1. Monitor CPU, memory, disk IOPS, and network throughput for 2+ weeks minimum
2. Identify VMs consistently below 30% utilization across all metrics
3. Recommend downsize to next smaller SKU in same family
4. Validate application performance after resize

### SKU Selection Guidelines
- **B-series (burstable)** — best for variable workloads with low average CPU (web servers, dev boxes)
- **D-series (general purpose)** — balanced CPU-to-memory, good default for most workloads
- **E-series (memory optimized)** — databases, caching, in-memory analytics
- **F-series (compute optimized)** — batch processing, gaming servers, high-CPU tasks
- **L-series (storage optimized)** — large local disk I/O, big data, data warehousing

### Common Mistakes
- Sizing for peak instead of P95 — use autoscaling for peak, right-size for baseline
- Ignoring memory metrics — VM may be CPU-idle but memory-constrained
- Not monitoring after resize — always validate application health post-change

## App Service Right-Sizing

### Assessment
- Monitor CPU and memory percentage in App Service plan metrics
- If consistently below 40% utilization, scale down to next lower tier
- Review instance count — reduce if autoscale rarely triggers scale-out

### Optimization Strategies
- **Per-app scaling** — run multiple apps on one plan, scale each independently
- **Deployment slots** — use for staging at no extra cost within the same plan
- **Free/Basic tiers for dev** — don't run dev/test workloads on Premium
- **Consolidate apps** — multiple low-traffic apps can share a single plan
- **Scale in during off-hours** — schedule reduced instance count nights/weekends

## Database Right-Sizing

### SQL Database
- **DTU vs vCore model** — DTU simpler for predictable workloads, vCore for fine-grained control
- **Elastic pools** — share resources across multiple databases with variable usage patterns
- **Serverless compute tier** — auto-pause after inactivity, pay only for compute used
- **Review service tier** — many databases run on Premium/Business Critical when General Purpose suffices

### Cosmos DB
- **Autoscale throughput** — automatically adjusts RU/s between min and max based on demand
- **Serverless** — ideal for dev/test and low-traffic workloads, no minimum RU charge
- **Manual throughput** — only for well-understood, stable workloads where autoscale overhead matters
- **Partition key design** — poor keys force over-provisioning to avoid hot partition throttling

## Container Right-Sizing

### Resource Requests and Limits
- Set CPU and memory requests based on actual observed usage, not defaults
- Requests determine scheduling — too high wastes node capacity, too low causes contention
- Limits prevent runaway containers — set 2x request as starting point, tune from there

### AKS Optimization
- **Vertical Pod Autoscaler (VPA)** — recommends optimal request/limit values based on historical usage
- **Cluster autoscaler** — scales node count based on pending pod scheduling
- **Node pool auto-scaling** — set min/max node counts per pool based on workload patterns
- **Right-size node SKU** — match node size to pod resource profiles to minimize wasted capacity

## Automation — Start/Stop Schedules

Non-production resources running 24/7 represent the easiest savings opportunity.

### Implementation
- **Azure Automation** — runbooks with schedules to start/stop VMs on a schedule
- **DevTest Labs** — built-in auto-shutdown and auto-start policies
- **Auto-shutdown** — configure at 7 PM local time for dev/test VMs
- **Auto-start** — configure at 8 AM local time on business days only

### Expected Savings
- 10 hours/day × 5 days/week = 50 hours running out of 168 total = **70% savings**
- Even a generous 12 hours/day × 7 days/week = 84 hours = **50% savings**
- Apply to all non-production VMs, AKS node pools, and App Service plans
- Use tags (e.g., `AutoShutdown=true`) to identify resources for scheduling
