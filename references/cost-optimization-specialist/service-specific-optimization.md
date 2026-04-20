# Service-Specific Cost Optimization

Each Azure service has unique cost drivers and optimization levers. This reference covers
the most impactful optimizations for commonly used services.

## Virtual Machines

VMs are typically the largest line item in Azure bills.

### Optimization Levers
- **Right-size** — downsize VMs consistently under 30% CPU/memory utilization
- **B-series burstable** — best for variable workloads with low average utilization
- **Spot VMs** — up to 90% savings for interruptible workloads (batch, CI/CD, dev/test)
- **Reserved Instances** — 30–72% savings for stable, predictable workloads
- **Savings Plans** — 15–65% savings with flexibility across sizes and regions
- **Auto-shutdown** — schedule dev/test VMs to stop at 7 PM, start at 8 AM
- **Azure Hybrid Benefit** — use existing Windows Server or SQL Server licenses (save 40–80%)

### Common Mistakes
- Running dev/test on D-series when B-series would suffice
- Not enabling Azure Hybrid Benefit when licenses are available
- Paying for Premium SSD on deallocated VMs — convert to Standard or delete disk

## Azure Kubernetes Service (AKS)

### Optimization Levers
- **Right-size node pools** — match node SKU to pod resource profiles
- **Cluster autoscaler** — scale node count based on pending pod demand
- **Node pool auto-scaling** — set min/max nodes per pool to avoid over-provisioning
- **Spot node pools** — use for interruptible workloads (batch jobs, CI agents)
- **Scale-to-zero** — for dev/test node pools that don't need to run continuously
- **Karpenter / Node Auto-Provisioning (NAP)** — optimized node scheduling, bin-packing
- **Vertical Pod Autoscaler (VPA)** — right-size pod CPU/memory requests automatically
- **AKS cost analysis** — native cost view by namespace, deployment, and node pool

### Common Mistakes
- Default resource requests/limits that waste 50%+ of node capacity
- Running a single large node pool instead of purpose-sized pools
- Not using spot pools for batch and CI workloads

## App Service

### Optimization Levers
- **Right-size the plan** — monitor CPU/memory, scale down if under 40% utilization
- **Free/B1 for dev** — don't run development workloads on Standard or Premium
- **Scale in during off-hours** — reduce instance count nights and weekends
- **Consolidate apps** — run multiple low-traffic apps on a single plan
- **Reserved Instances** — for production plans with stable, predictable usage
- **Per-app scaling** — scale individual apps independently on shared plans

### Common Mistakes
- Premium tier for a single low-traffic app
- Deployment slots running idle on expensive plans
- Always On enabled on Basic tier (wastes money, provides minimal benefit)

## SQL Database

### Optimization Levers
- **Serverless compute tier** — auto-pause after configurable inactivity period, ideal for intermittent use
- **Elastic pools** — share DTU/vCore resources across multiple databases with variable usage
- **Azure Hybrid Benefit** — use existing SQL Server licenses for up to 55% savings
- **Reserved capacity** — 1-year or 3-year commitment for stable DTU/vCore workloads
- **Short-term backup retention** — reduce from default 7 days if recovery needs are lower
- **General Purpose vs Business Critical** — many databases don't need Business Critical tier

### Common Mistakes
- Every database on its own dedicated compute when elastic pools would save 30–50%
- Business Critical tier for workloads that don't need in-memory OLTP or low-latency reads
- Not enabling Hybrid Benefit when SQL licenses are available through EA

## Storage

### Optimization Levers
- **Lifecycle management policies** — automatically tier blobs from Hot → Cool → Cold → Archive based on age
- **Delete old snapshots and versions** — set retention policies to prevent unlimited growth
- **Reserved capacity** — 1-year commitment for predictable Hot or Cool storage volumes
- **ADLS Gen2 reserved capacity** — for data lake workloads with stable storage needs
- **Correct redundancy** — use LRS for dev/test, don't pay for GRS on non-critical data
- **Correct access tier** — data accessed less than once per month should be Cool or Cold

### Common Mistakes
- All data on Hot tier regardless of access patterns
- No lifecycle policy — data grows indefinitely at Hot tier pricing
- GRS redundancy on dev/test or ephemeral data

## Cosmos DB

### Optimization Levers
- **Autoscale throughput** — automatically adjusts RU/s between 10% min and configured max
- **Serverless** — ideal for dev/test and low-traffic workloads, pay per request
- **Reserved capacity** — up to 65% savings on stable provisioned RU/s workloads
- **Optimize partition key** — well-designed keys distribute load evenly, reducing RU overprovisioning
- **Indexing policy** — exclude unnecessary paths from indexing to reduce write RU cost
- **Multi-region** — only replicate to regions where low-latency reads are required

### Common Mistakes
- Default indexing policy that indexes every property (doubles write RU cost)
- Provisioned throughput for dev/test when serverless is cheaper
- Manual throughput without monitoring — over-provisioning for safety wastes 50%+

## Networking

### Optimization Levers
- **Remove unused public IPs** — $3.65/month each, hundreds across an enterprise
- **VPN Gateway Basic SKU** — sufficient for dev/test, avoid VpnGw1+ when not needed
- **NAT Gateway shared across subnets** — one gateway serves multiple subnets, avoid per-subnet cost
- **ExpressRoute metered vs unlimited** — choose based on actual bandwidth consumption
- **Private Endpoint consolidation** — group services to minimize per-endpoint charges
- **Reduce cross-region egress** — co-locate resources that communicate frequently

### Common Mistakes
- ExpressRoute Premium when Standard meets connectivity needs (~$4K/month difference)
- Azure Firewall running with minimal traffic ($900/month base regardless of usage)
- Application Gateway WAF v2 with no backends (~$250/month wasted)
