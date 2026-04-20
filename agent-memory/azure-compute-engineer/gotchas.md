# Azure Compute Gotchas

Surprising behaviors, hidden limits, and "read the fine print" issues across Azure compute services.

## Virtual Machines

| Gotcha | Detail | Workaround |
|--------|--------|------------|
| **VM resizing may require deallocation** | Some size changes (e.g., crossing families like B→D, or changing accelerated networking support) require stop/deallocate first — causes downtime | Plan resize during maintenance windows; use VMSS with rolling upgrades for zero-downtime size changes |
| **Accelerated networking not available on all sizes** | Only supported on VMs with 2+ vCPUs; not available on A-series, basic-tier, or some older sizes | Check `az vm list-sizes` with `--query` for `acceleratedNetworkingEnabled` before selecting SKU |
| **Ephemeral OS disks lost on deallocation** | Faster boot times and no storage cost, but OS disk state is destroyed on stop/deallocate | Use only for stateless workloads; store state externally (Azure Files, blob, managed disks) |
| **Spot VM eviction: 30-second notice** | Azure reclaims capacity with only ~30 seconds warning via Scheduled Events | Poll Scheduled Events endpoint; implement graceful shutdown and checkpoint logic |
| **Reserved instances locked to region and VM size family** | Cannot move RI between regions; limited to size flexibility within same family | Use savings plans for cross-region/cross-family flexibility; RIs only for well-understood stable workloads |
| **Auto-shutdown ≠ deallocate by default** | Auto-shutdown stops the OS but may not deallocate (depends on configuration) — you still pay for the allocated VM | Ensure auto-shutdown is configured to deallocate; verify in billing |
| **Proximity placement groups limit availability** | PPGs reduce latency but constrain placement — can cause allocation failures in capacity-constrained regions | Use only for latency-sensitive tiers; don't apply PPGs to entire deployments |

## Virtual Machine Scale Sets

| Gotcha | Detail | Workaround |
|--------|--------|------------|
| **Uniform vs Flexible mode: different behavior** | Uniform: identical instances, classic autoscale. Flexible: mixed sizes, AZ-aware, integrates with availability zones natively. Behaviors differ for networking, placement, and extensions | New deployments should prefer Flexible mode; Uniform is legacy for most scenarios |
| **Autoscale cooldown matters** | Default cooldown is 5 minutes — rapid scale-in after scale-out can oscillate and cause instability | Tune cooldown periods; set scale-in to be slower than scale-out (e.g., 10 min in, 5 min out) |
| **Image updates don't auto-apply to existing instances** | Updating the VMSS model image doesn't reimage running instances | Use rolling upgrade policy or manual `az vmss update-instances` |

## Azure Kubernetes Service

| Gotcha | Detail | Workaround |
|--------|--------|------------|
| **Node pool VM size cannot be changed after creation** | You cannot resize VMs in an existing node pool — you must create a new pool, cordon/drain, delete old | Plan VM sizing carefully; use multiple node pools from the start for flexibility |
| **Cluster autoscaler scale-up takes minutes, not seconds** | New nodes need VM provisioning + image pull + kubelet registration — typically 2-5 minutes | Use KEDA for app-level scaling; consider virtual nodes (ACI) for burst; keep minimum node count adequate |
| **System node pool requirements** | Must have at least 1 system node pool; certain addons (e.g., monitoring, policy) require specific SKUs or minimum resources | Don't try to remove all system pools; use Standard_DS2_v2 or larger for system pools |
| **AKS API throttling at subscription level** | All AKS clusters in a subscription-region share Azure API limits — too many clusters or clients causes throttling | Limit to 20-40 clusters per subscription-region; upgrade to latest K8s version for optimized API calls |
| **Pod CIDR default is 10.244.0.0/16** | Each node gets a /24 — limits practical max to ~250 pods per node (Kubernetes recommends ≤110) | Plan pod CIDR based on expected cluster scale; use `--max-pods` setting |
| **Windows node pools have limitations** | No support for Calico network policies, some CSI drivers, and certain node features | Check [AKS Windows limitations](https://learn.microsoft.com/azure/aks/windows-faq) before committing |

## Container Apps

| Gotcha | Detail | Workaround |
|--------|--------|------------|
| **4 vCPU / 8 GB max per container** | Hard limit on Consumption plan — cannot exceed per-container | Use Dedicated plan (workload profiles) for larger containers; or split workload across multiple containers |
| **No Windows container support** | Container Apps is Linux-only | Use AKS for Windows containers |
| **Revision management can be confusing** | Multiple active revisions with traffic splitting — inactive revisions still consume resources if not deactivated | Explicitly deactivate old revisions; use single-revision mode for simpler apps |
| **Cold start on Consumption plan** | First request after idle triggers container image pull — larger images = longer cold start | Keep images small (<500MB); use Dedicated plan for latency-sensitive workloads |

## Azure Functions

| Gotcha | Detail | Workaround |
|--------|--------|------------|
| **Consumption plan: 10-minute max execution** | Functions timeout after 10 minutes on Consumption — silently killed | Use Durable Functions for orchestration; Premium/Dedicated plan for long-running (up to 60 min) |
| **Consumption plan: 1.5 GB memory limit** | Cannot increase memory on Consumption plan | Use Premium (up to 14 GB) or Flex Consumption for memory-intensive functions |
| **Cold start varies by language** | .NET in-process: ~1s; Java: 5-10s; Python: 2-5s on Consumption | Use always-ready instances (Flex Consumption) or pre-warmed instances (Premium) |
| **Flex Consumption: Linux only** | No Windows support on Flex Consumption plan | Use Premium plan for Windows Functions workloads |

## App Service

| Gotcha | Detail | Workaround |
|--------|--------|------------|
| **Too many apps on one plan = resource contention** | All apps in a plan share the same underlying VM instances — one noisy app degrades all | Monitor per-app CPU/memory; isolate high-traffic apps on dedicated plans |
| **Deployment slots share plan resources** | Staging slot uses same compute as production — load testing staging impacts prod | Scale up plan temporarily during slot-based load testing |
| **Free/Shared tiers: no VNet integration, no custom domains (Free), no SLA** | Often used for production by mistake | Use Basic B1 minimum for any real workload; Standard for VNet integration |
| **ARR affinity enabled by default** | Sticky sessions can cause uneven load distribution across instances | Disable ARR affinity for stateless apps; use external session state (Redis) |

## Cross-Cutting Gotchas

| Gotcha | Detail | Workaround |
|--------|--------|------------|
| **Azure subscription throttling** | All compute in a subscription-region shares API rate limits — too many VMSS/AKS clusters cause 429 errors | Spread large deployments across subscriptions; use latest API versions |
| **Resource locks prevent scaling** | A `ReadOnly` lock on a resource group blocks VMSS autoscale operations | Use `CanNotDelete` lock instead if you need autoscale; apply locks per-resource |
| **Managed identity propagation delay** | After assigning managed identity, RBAC role assignment can take 5-10 minutes to propagate | Build retry logic into startup scripts; don't fail hard on first auth attempt |
| **Azure region capacity constraints** | Not all VM sizes available in all regions — certain SKUs frequently sold out in popular regions | Have fallback regions and VM sizes; check availability with `az vm list-skus` |
| **Changing VM availability zone requires redeployment** | Cannot move a VM between zones after creation | Plan zone placement before deployment; use VMSS Flexible for cross-zone distribution |

## Quick "Did You Know?" List

- VMSS Flexible mode is the new default — Uniform mode is legacy and will not receive new features
- AKS node surge during upgrades defaults to 1 extra node — increase `max-surge` for faster upgrades at the cost of temporary extra capacity
- Container Apps environment-level VNet injection is set at creation — cannot change VNet after
- Functions Durable entities have a 1 MB serialization limit per entity — large state must be externalized
- App Service "Always On" is required for WebJobs and background tasks — disabled by default on Basic tier

## References

- [Azure Spot Virtual Machines — eviction policy](https://learn.microsoft.com/azure/virtual-machines/spot-vms)
- [Resize node pools in AKS](https://learn.microsoft.com/azure/aks/resize-node-pool)
- [Architecture best practices for VMs](https://learn.microsoft.com/azure/well-architected/service-guides/virtual-machines)
- [Build workloads on spot virtual machines](https://learn.microsoft.com/azure/architecture/guide/spot/spot-eviction)
