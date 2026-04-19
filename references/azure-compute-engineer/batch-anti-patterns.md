## Azure Batch

### When to Use Batch
- Embarrassingly parallel workloads (image rendering, Monte Carlo, genomics)
- Large-scale parallel and HPC workloads
- Jobs that don't need real-time response
- Pre-defined start and finish (not long-running services)

### Key Concepts
- **Pool**: Collection of VMs (nodes) — auto-scale or fixed
- **Job**: Collection of tasks assigned to a pool
- **Task**: Single unit of computation
- **Low-priority/Spot nodes**: Up to 80% discount for fault-tolerant workloads
- **Scaling**: Auto-scale formula based on queue depth, CPU, time-of-day

## Common Mistakes and Anti-Patterns

1. **B-series for sustained workloads** → CPU throttled once credits exhausted → mysterious slowness
2. **Oversizing VMs "just in case"** → 80%+ of VMs run at < 20% CPU → enormous waste
3. **Reserved Instances before right-sizing** → locked into wrong sizes for 3 years
4. **AKS without cluster autoscaler** → paying for idle nodes 24/7
5. **Functions Consumption for latency-sensitive APIs** → cold starts ruin P99 latency
6. **AKS for 3 microservices** → massive operational overhead for a small workload → Container Apps
7. **Single node pool for everything** → can't right-size, can't use spot for batch, can't isolate system pods
8. **Ignoring App Service Plan density** → one app per plan = expensive; pack multiple apps per plan (but monitor)
9. **Not setting pod resource requests/limits** → scheduler can't bin-pack → wasted capacity → OOM kills
10. **Manual scaling in production** → weekend traffic spike → no one's watching → outage

## Questions This Specialist Always Asks

### Workload Profiling
- What's the workload type? (Web/API, batch, event-driven, long-running service, HPC?)
- Traffic pattern? (Steady, bursty, predictable peaks, seasonal?)
- Latency requirements? (Sub-second? Seconds acceptable? Minutes for batch?)
- Stateful or stateless? (Affects scaling strategy, spot viability)

### Existing Stack
- Containerized already? (If not, App Service code deploy is fastest path)
- Kubernetes expertise on the team? (If no → don't start with AKS)
- Existing App Service Plans with headroom? (Pack before creating new)
- Current VM inventory? (Right-size before committing reserved capacity)

### Cost & Commitment
- Budget constraints? (Determines tier selection and commitment strategy)
- Commitment horizon? (1 year, 3 year, or unknown → drives RI vs Savings Plan vs PAYG)
- Fault tolerance? (Drives spot instance viability)
- Dev/test vs production? (Dev/test pricing, B-series, lower SLA tiers)

### Scale Requirements
- Min/max instance count? (Determines autoscale bounds and commitment)
- Scale-to-zero needed? (Functions Consumption, Container Apps, or KEDA)
- Cold start acceptable? (If not → Premium Functions, always-ready Container Apps, or persistent VMs)
- Multi-region? (Traffic Manager, Front Door, AKS Fleet Manager)
