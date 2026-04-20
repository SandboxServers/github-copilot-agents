# Azure Batch & Anti-Patterns

## Azure Batch Overview

Managed service for running large-scale parallel and batch compute jobs without managing infrastructure.

**Use Azure Batch for:** Rendering, simulation, genomics, financial modeling, Monte Carlo, transcoding, large-scale data processing — any embarrassingly parallel workload.

### Architecture

```
Azure Batch Account
├─ Pool (collection of VMs / compute nodes)
│  ├─ Fixed pool: pre-provisioned, always available
│  ├─ Auto-scale pool: formula-based (queue depth, time, CPU)
│  └─ Auto-pool: created per job, destroyed after job completes
├─ Job (logical grouping of tasks, assigned to a pool)
│  ├─ Task 1 (single unit of computation)
│  ├─ Task 2
│  └─ Task N
└─ Job Schedule (optional — recurring jobs on a schedule)
```

### Key Features

| Feature | Description |
|---------|------------|
| **Auto-pool** | Create and destroy pool per job — no idle VMs between jobs |
| **Low-priority / Spot nodes** | Up to 80-90% discount for fault-tolerant tasks |
| **Task dependencies** | Chain tasks: Task B runs only after Task A completes |
| **Multi-instance tasks** | MPI workloads across multiple nodes |
| **Container support** | Run tasks in Docker containers |
| **Start tasks** | Setup script runs on each node (install dependencies, mount storage) |
| **Application packages** | Upload ZIP packages, deployed to nodes automatically |
| **Managed identity** | Pool-level managed identity for accessing Azure resources |

### Cost Optimization for Batch

- **Auto-pool:** Create pool at job start, delete at job end — zero cost between jobs
- **Low-priority/Spot nodes:** Use for all fault-tolerant tasks. Implement checkpointing for long tasks
- **Right-size pools:** Use auto-scale formulas based on pending task count, not fixed-size pools
- **Choose efficient VM sizes:** F-series for CPU-bound, E-series for memory-bound, N-series for GPU
- **Task slots per node:** Configure multiple task slots per node to maximize utilization (e.g., 4 tasks on 4-vCPU VM)

## Compute Anti-Patterns

| Anti-Pattern | Problem | Better Approach |
|-------------|---------|-----------------|
| **AKS for simple batch jobs** | Massive operational overhead for periodic processing | Azure Batch (auto-pool) or Functions with timer trigger |
| **VMs for periodic processing** | Paying for idle VMs between runs, manual scheduling | Azure Batch (auto-pool) or Functions with timer trigger |
| **Single large VM for parallel work** | No fault tolerance, limited by single machine, poor utilization | Batch pool of smaller VMs or VMSS with autoscale |
| **No autoscaling configured** | Paying for peak capacity 24/7 | Right-size with auto-pool, VMSS autoscale, or AKS cluster autoscaler |
| **Premium compute for dev/test** | Production-grade VMs sitting mostly idle | B-series burstable with auto-shutdown |
| **Ignoring Spot/Low-Priority VMs** | Paying full price for interruptible workloads | Use Spot for batch, dev/test, CI/CD (up to 90% savings) |
| **AKS for a single service** | Cluster overhead ($300+/month minimum) for one container | Container Apps or App Service |
| **Functions with 10-min+ runtime** | Consumption plan timeout, cold starts, unreliable for long ops | Durable Functions, Container Apps Jobs, or Azure Batch |
| **B-series for sustained workloads** | CPU credit exhaustion → throttled to baseline → mystery slowness | D-series general purpose or F-series compute-optimized |
| **Reserved Instances before right-sizing** | Locked into wrong VM size for 1-3 years | Right-size for 2+ weeks, THEN commit |
| **Functions Premium for rare triggers** | $155+/month minimum for a function that runs hourly | Consumption or Flex Consumption |
| **Over-provisioned App Service Plan** | One P3v3 ($330/month) running a WordPress blog | Basic B1 ($55) or pack multiple apps per Standard plan |
| **AKS without cluster autoscaler** | Nodes running 24/7 at 15% utilization | Enable cluster autoscaler with min/max per node pool |
| **Ignoring Container Apps** | Defaulting to AKS for "containers" when K8s isn't needed | Container Apps for most containerized microservices |
| **Manual scaling in production** | Weekend traffic spike, no one watching → outage | Autoscale rules with appropriate min/max and alerts |
| **GPU VMs running idle** | N-series VMs at $1-10+/hour sitting idle between training runs | Spot GPU VMs, AKS with GPU node pool autoscaler, scale-to-zero |

## Decision Shortcuts

**When someone says "we need VMs for batch processing":**
→ Ask: Is it embarrassingly parallel? → Azure Batch with auto-pool
→ Ask: Is it a periodic scheduled task? → Functions timer trigger or Container Apps Job
→ Ask: Does it need GPU? → AKS GPU node pool with spot VMs or Batch with N-series pool

**When someone says "we need Kubernetes":**
→ Ask: Why specifically K8s? Custom operators? Service mesh? Windows containers?
→ If no: Container Apps covers 80% of container workloads with 20% of the complexity

**When someone says "it needs to be always running":**
→ Ask: Does it actually? Or can it respond to events?
→ Scale-to-zero (Functions/Container Apps) eliminates cost for idle workloads
