## Cost Optimization Levers

### Reserved Instances vs Savings Plans vs Spot vs PAYG

| Option | Discount | Commitment | Flexibility | Best For |
|--------|----------|-----------|-------------|----------|
| **Pay-as-you-go** | 0% | None | Full | Short-term, unpredictable workloads |
| **Reserved Instances (RI)** | Up to 72% | 1 or 3 year | Locked to specific VM size + region | Steady-state VMs, known sizing |
| **Savings Plans** | Up to 65% | 1 or 3 year $/hr | Any VM family, any region | Diverse compute, changing sizes |
| **Spot Instances** | Up to 90% | None (can be evicted) | Size available at discount | Batch, HPC, fault-tolerant, dev/test |
| **Dev/Test pricing** | ~50% | MSDN/VS subscription | Windows VMs only | Dev/test environments |

### Spot Instance Rules
- **Can be evicted at any time** when Azure needs capacity (30-second notice)
- **Set max price** or use -1 for market price (up to PAYG)
- **Eviction policies**: Stop/Deallocate (can restart later) or Delete
- **Good for**: Batch processing, CI/CD build agents, dev/test, stateless workers, ML training with checkpointing
- **Never for**: Production web servers, databases, anything requiring uptime SLA

### Right-Sizing Strategy
```
Step 1: Enable Azure Advisor → identify underutilized VMs (CPU < 5% avg)
Step 2: Check 30-day metrics: CPU, memory, network, disk IOPS
Step 3: Common findings:
  ├─ D8s_v5 running at 10% CPU → downsize to D4s_v5 or D2s_v5
  ├─ E-series with low memory usage → move to D-series
  ├─ Steady 40% CPU 24/7 → perfect candidate for Reserved Instance
  ├─ Burst to 80% for 2 hours/day → consider B-series (if baseline fits)
  └─ GPU VM idle 20 hours/day → scale-to-zero or spot instance
Step 4: Right-size, then commit (RI or Savings Plan) — never commit first
```
