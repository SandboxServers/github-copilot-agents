# Compute Cost Optimization

## Commitment Discounts

| Option | Discount | Commitment | Flexibility | Best For |
|--------|----------|-----------|-------------|----------|
| **Pay-as-you-go (PAYG)** | 0% | None | Full — stop anytime | Short-term, unpredictable, experimentation |
| **Reserved Instances (RI)** | Up to 72% | 1 or 3 year | Locked to specific VM size + region (exchangeable within family) | Steady-state VMs with known sizing |
| **Savings Plans** | Up to 65% | 1 or 3 year $/hr commitment | Any VM family, any region, any OS | Diverse compute, changing sizes, multi-region |
| **Spot VMs** | Up to 90% | None (evictable at any time) | Size available at discount, 30-second eviction notice | Batch, HPC, fault-tolerant, dev/test, ML training |
| **Dev/Test pricing** | ~50% on Windows VMs | Visual Studio subscription | Windows VMs only, no SLA | Dev/test environments |

### Reserved Instances vs Savings Plans

| Factor | Reserved Instances | Savings Plans |
|--------|-------------------|---------------|
| **Scope** | Specific VM size + region | Any compute across regions |
| **Flexibility** | Exchange within same family, cancel with fee | Fully flexible (any VM, App Service, Functions Premium) |
| **Max discount** | 72% (3-year) | 65% (3-year) |
| **Best for** | Known, stable workloads you won't change | Dynamic environments, multi-service, multi-region |
| **Risk** | Locked if workload changes | Lower discount but no lock-in risk |

**Strategy:** Use RIs for stable baseline (databases, core infrastructure). Use Savings Plans for dynamic compute. Use PAYG for spiky or experimental workloads.

## Spot VM Rules

- **Eviction:** Azure can reclaim at any time with 30-second notice. Design for graceful shutdown
- **Max price:** Set explicit max or -1 for market price up to PAYG rate
- **Eviction policies:** Stop/Deallocate (restart later) or Delete
- **Good for:** Batch processing, CI/CD build agents, dev/test, stateless workers, ML training with checkpointing
- **Never for:** Production web servers, databases, anything requiring uptime SLA
- **Availability varies:** Popular sizes in popular regions may have limited spot capacity

## Right-Sizing Strategy

```
Step 1: Deploy with initial sizing estimate
Step 2: Monitor 2+ weeks — collect CPU, memory, disk IOPS, network
Step 3: Analyze p95 metrics (not averages):
  ├─ D8s_v5 at 10% avg CPU → D4s_v5 or D2s_v5 (50-75% savings)
  ├─ E-series with low memory → D-series (memory-optimized unnecessary)
  ├─ Steady 40% CPU 24/7 → strong RI/Savings Plan candidate
  ├─ Spikes to 80% for 2 hrs/day → B-series (if baseline fits)
  └─ GPU VM idle 20 hrs/day → scale-to-zero or spot instance
Step 4: Right-size FIRST, then commit (RI or Savings Plan)
Step 5: Re-evaluate quarterly — workloads change, new SKUs launch
```

**Never commit before right-sizing** — buying a 3-year RI on an oversized VM locks in waste.

## Scale-to-Zero for Cost Elimination

| Service | Idle Cost | How |
|---------|-----------|-----|
| **Functions Consumption** | $0 | Automatic — no instances when no events |
| **Functions Flex Consumption** | $0 (or always-ready cost) | Configurable always-ready count |
| **Container Apps Consumption** | $0 | Scale rule min replicas = 0 |
| **VMSS** | $0 (lose state) | Autoscale rule with min = 0 |
| **AKS (KEDA)** | Node cost remains | Pods scale to zero, but nodes don't auto-deprovision below min |

## Dev/Test Cost Savings

- **Azure Dev/Test pricing** — up to 50% off Windows VMs via Visual Studio subscription
- **Auto-shutdown VMs** — schedule shutdown at 7 PM, start at 8 AM (save ~60% on 24/7 cost)
- **B-series burstable** — cheapest VMs for dev/test (B1s at ~$8/month)
- **Smaller SKUs** — dev doesn't need production-grade E64s_v5
- **Spot VMs** — dev/test is inherently interruptible
- **Azure Dev/Test Labs** — automated provisioning with auto-shutdown policies and cost limits

## AKS-Specific Cost Optimization

| Strategy | Savings | Trade-off |
|----------|---------|-----------|
| **Spot node pools** | Up to 90% on batch nodes | Eviction risk — only for interruptible workloads |
| **Cluster autoscaler** | Avoid idle nodes | Scale-up latency (minutes for new nodes) |
| **NAP (Node Auto Provisioning)** | Optimal VM selection | Less control over specific VM SKUs |
| **Scale down off-hours** | 50-70% on dev/test clusters | Requires schedule-based autoscaler or external automation |
| **Right-size node pools** | 30-50% | Requires monitoring and periodic adjustment |
| **Start/Stop cluster** | 100% (stopped = free, except disk) | Downtime — only for dev/test |
| **Pod resource requests** | Better bin-packing = fewer nodes | Requires accurate resource estimates |

## Common Cost Waste

| Waste Pattern | Estimated Waste | Fix |
|--------------|----------------|-----|
| Over-provisioned App Service Plan (P3v3 for a blog) | $300+/month | Downsize to B1 or S1, pack multiple apps |
| Premium Functions rarely executing | $155+/month baseline | Switch to Consumption or Flex Consumption |
| Idle AKS cluster (dev/test running 24/7) | $300+/month minimum | Auto-shutdown, Start/Stop cluster, spot nodes |
| Orphaned managed disks after VM deletion | $5-100+/month each | Audit with Azure Resource Graph, delete unused |
| Orphaned public IPs after resource deletion | $4/month each | Tag resources, audit regularly |
| Unused Reserved Instances (workload decommissioned) | Full RI cost | Exchange for different size, cancel (with fee) |
| AKS without cluster autoscaler | 30-60% waste on idle nodes | Enable autoscaler with appropriate min/max |
| Single large VM for parallel workloads | Poor utilization + no fault tolerance | Azure Batch pool of smaller VMs, or VMSS |
| Production VMs without commitment discount | 40-72% premium over RI pricing | Analyze steady-state, apply RI or Savings Plan |

## Cost Monitoring Tools

| Tool | Purpose |
|------|---------|
| **Azure Cost Management** | Budget alerts, cost analysis, export to storage |
| **Azure Advisor** | Right-sizing recommendations, unused resource detection |
| **Azure Resource Graph** | Query all resources for orphaned disks, IPs, snapshots |
| **Cost alerts** | Budget thresholds, anomaly detection, forecast alerts |
| **Tags** | Cost allocation by team, project, environment — enforce via Azure Policy |
