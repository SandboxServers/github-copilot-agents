# Azure Compute Anti-Patterns

Common mistakes in Azure compute architecture that lead to wasted spend, poor performance, or unnecessary operational burden.

## Sizing & Provisioning Anti-Patterns

| Anti-Pattern | Why It Happens | Impact | Better Approach |
|-------------|----------------|--------|-----------------|
| **Over-provisioning VMs 2-3x** | "Just in case" mindset from on-prem | 50-70% wasted spend on idle capacity | Right-size based on 2+ weeks of actual CPU/memory metrics via Azure Monitor |
| **Under-provisioning** | Optimizing cost without load testing | OOM kills, CPU throttling, cascading failures | Use Azure Migrate assessments or Monitor Insights before committing to size |
| **B-series for sustained workloads** | Choosing cheapest option blindly | CPU credit exhaustion → throttled to baseline (20% of vCPU) | B-series only for intermittent workloads (dev boxes, low-traffic APIs); use D/F-series for sustained |
| **Ignoring burstable B-series for intermittent work** | Defaulting to D-series for everything | Paying for constant compute on workloads that idle 80%+ of the time | B-series is ideal for dev/test, build agents, low-traffic web apps |
| **Reserved Instances before right-sizing** | Locking in savings too early | Committed to wrong VM size for 1-3 years, can't change | Right-size for 2+ weeks with monitoring, THEN buy RI or savings plan |
| **Not reviewing sizing after initial deployment** | "Set and forget" culture | Persistent over/under-provisioning as workload patterns change | Schedule quarterly right-sizing reviews using Azure Advisor recommendations |

## Availability & Resilience Anti-Patterns

| Anti-Pattern | Why It Happens | Impact | Better Approach |
|-------------|----------------|--------|-----------------|
| **Single-instance VMs without zones or availability sets** | "It's just one server" | No SLA for single instances without premium storage; zone failure = total outage | Use availability zones for production; availability sets minimum |
| **Not using VMSS when manual scaling is a recurring pain** | Avoiding migration effort | Repeated manual intervention, slow response to load spikes, human error | VMSS with autoscale rules based on actual metrics |
| **Running Spot VMs without eviction handling** | Treating spot like regular VMs | Workload killed mid-execution with only 30-second notice, data loss | Implement checkpointing, use eviction-aware orchestration, design for restart |

## Service Selection Anti-Patterns

| Anti-Pattern | Why It Happens | Impact | Better Approach |
|-------------|----------------|--------|-----------------|
| **AKS when Container Apps would suffice** | "We need Kubernetes" without evaluating needs | $300+/month minimum cluster cost, operational overhead for ingress/certs/upgrades | Container Apps handles 80% of container workloads with Dapr, KEDA, managed TLS |
| **AKS for a single microservice** | Defaulting to K8s for any container | Over-engineered infrastructure for simple workloads | App Service or Container Apps for single services |
| **Functions Consumption for latency-sensitive workloads** | Scale-to-zero appeal | Cold starts of 1-10 seconds on first invocation, unacceptable for real-time APIs | Functions Premium (pre-warmed) or Flex Consumption (always-ready instances) |
| **Functions Premium for rarely-triggered functions** | Avoiding cold start at any cost | $155+/month minimum for a function that runs a few times per hour | Consumption plan is fine if cold start is tolerable; Flex Consumption for middle ground |
| **VMs for periodic batch processing** | Familiar pattern from on-prem | Paying for idle VMs between runs | Azure Batch (auto-pool), Container Apps Jobs, or Functions timer triggers |

## Cost Anti-Patterns

| Anti-Pattern | Why It Happens | Impact | Better Approach |
|-------------|----------------|--------|-----------------|
| **Dev/test workloads running 24/7** | No one set up auto-shutdown | Paying for 128+ hours/week when 40 hours used | Enable auto-shutdown on dev VMs; use DevTest Labs; schedule-based VMSS scaling |
| **Not using Spot VMs for fault-tolerant workloads** | Fear of eviction complexity | Paying full price for batch, dev/test, CI/CD agents | Spot VMs save up to 90%; suitable for batch, dev/test, stateless processing |
| **No reserved instances or savings plans for stable workloads** | "We might change sizes" indefinitely | Paying on-demand rates for workloads that haven't changed in 6+ months | Savings plans (flexible across sizes/regions) or RIs for predictable workloads |
| **GPU VMs running idle between training runs** | No scale-to-zero for GPU | N-series at $1-10+/hour sitting idle | AKS GPU node pool with cluster autoscaler (scale to 0), or Spot GPU VMs |

## AKS-Specific Anti-Patterns

| Anti-Pattern | Why It Happens | Impact | Better Approach |
|-------------|----------------|--------|-----------------|
| **Single node pool for all workload types** | Simplicity over optimization | CPU-heavy and memory-heavy workloads competing; GPU waste on non-GPU pods | Separate node pools: system pool, general workload pool, GPU pool, spot pool |
| **No cluster autoscaler enabled** | "We sized it right at launch" | Nodes at 15% utilization, paying for peak 24/7 | Enable cluster autoscaler with appropriate min/max per node pool |
| **AKS without pod resource requests/limits** | "It works without them" | Noisy neighbor problems, OOM kills, unpredictable scheduling | Always set CPU/memory requests and limits; use LimitRange for defaults |

## Networking & Configuration Anti-Patterns

| Anti-Pattern | Why It Happens | Impact | Better Approach |
|-------------|----------------|--------|------------------|
| **Public IPs on every VM** | Quick-and-dirty access | Attack surface, compliance violations, cost per IP | Use Azure Bastion, VPN, or private endpoints; NSGs as defense-in-depth |
| **No health probes on load-balanced VMs** | Assuming LB "just works" | Traffic routed to unhealthy instances → user-facing errors | Configure TCP or HTTP health probes on every backend pool |
| **Skipping managed identity** | Using connection strings / keys | Credential rotation burden, secrets in config, breach risk | Managed identity on all compute; Key Vault for any remaining secrets |
| **Same App Service Plan for all environments** | Simplifying infrastructure | Dev deployments impact production performance | Separate plans per environment; use deployment slots within prod plan only |

## Quick Smell Tests

- **"We need 64 vCPUs because our on-prem server had that"** → Profile actual usage first; cloud workloads rarely mirror on-prem sizing
- **"Let's use AKS, we're doing containers"** → Do you need custom operators, service mesh, or Windows containers? If no, start with Container Apps
- **"It needs to run 24/7"** → Does it actually? Event-driven scale-to-zero eliminates cost for idle workloads
- **"We'll right-size later"** → Later never comes. Set a calendar reminder for 2 weeks post-deployment
- **"We'll just use the default VM size"** → Defaults are general purpose; check if workload is CPU-bound, memory-bound, or IO-bound
- **"Spot VMs are too risky"** → For batch, dev/test, and CI/CD they're ideal. Build eviction handling, not avoidance
- **"We don't need autoscale, we know our traffic"** → You know average traffic. You don't know Black Friday, viral events, or DDoS

## Decision Anti-Pattern Flowchart

```
Team says "we need VMs"
├─ Why? → "That's what we always used" → ANTI-PATTERN: Evaluate PaaS first
├─ Why? → "We need GPU" → OK, but use dedicated GPU node pool, not standalone VMs
├─ Why? → "Custom OS/software" → VM is correct, but use VMSS + autoscale, not single VM
└─ Why? → "Batch processing" → Use Azure Batch auto-pool or Container Apps Jobs
```

## References

- [Performance antipatterns for cloud applications](https://learn.microsoft.com/azure/architecture/antipatterns/)
- [FinOps best practices for compute](https://learn.microsoft.com/cloud-computing/finops/best-practices/compute)
- [Architecture best practices for VMs and scale sets](https://learn.microsoft.com/azure/well-architected/service-guides/virtual-machines)
