## AKS Deep Dive

### Node Pool Strategy

| Pool Type | Purpose | Configuration |
|-----------|---------|---------------|
| **System pool** | CoreDNS, kube-proxy, metrics-server | Small (3 nodes), Standard_D2s_v5, taint: CriticalAddonsOnly |
| **General workload pool** | Application pods | D-series, autoscaler enabled, min 2 / max 20 |
| **Memory-intensive pool** | Databases, caches | E-series, node labels for affinity |
| **Spot pool** | Batch jobs, non-critical workloads | Spot VMs, taint: kubernetes.azure.com/scalesetpriority=spot |
| **GPU pool** | ML training/inference | N-series, taint to prevent non-GPU pods |

### AKS Networking Models

| Model | CNI | IP Management | Best For |
|-------|-----|--------------|----------|
| **Kubenet** | Basic | Pod IPs from virtual pod CIDR, NAT to node IP | Small clusters, IP-constrained networks |
| **Azure CNI** | Advanced | Pod IPs from VNet subnet | VNet integration, NSG per pod, large subnets |
| **Azure CNI Overlay** | Advanced + overlay | Pod IPs from overlay CIDR | Large clusters, IP conservation |
| **Azure CNI + Cilium** | eBPF-based | Pod IPs from VNet or overlay | Network policy enforcement, observability |

### AKS Upgrade Strategy
- **Automatic channel**: None, Patch, Stable, Rapid, Node Image
- **Blue-green**: Create new node pool → cordon old → migrate pods → drain old pool
- **Maintenance windows**: Schedule upgrade windows to avoid business hours
- **PDB (Pod Disruption Budget)**: Always set — prevents upgrades from killing all pods at once
- **Surge upgrades**: Configure `max-surge` to add extra nodes during upgrade (speed vs cost trade-off)
