# AKS Deep Dive

## Cluster Architecture

AKS separates the **control plane** (managed by Azure, free) from **node pools** (your VMs, your cost).

```
AKS Cluster
├─ Control Plane (managed by Azure — free tier or Standard/Premium tier)
│  ├─ API Server (private or public endpoint)
│  ├─ etcd (cluster state)
│  ├─ Scheduler
│  └─ Controller Manager
├─ System Node Pool (required — runs K8s system components)
│  ├─ CoreDNS
│  ├─ kube-proxy
│  └─ metrics-server
└─ User Node Pools (your workloads)
   ├─ General pool (D-series, autoscaled)
   ├─ Memory pool (E-series, labeled for affinity)
   ├─ Spot pool (D-series Spot, tainted)
   └─ GPU pool (N-series, tainted)
```

## Node Pool Strategy

| Pool Type | Purpose | Recommended Config |
|-----------|---------|-------------------|
| **System pool** | CoreDNS, kube-proxy, metrics-server, konnectivity | 3x D2s_v5, taint `CriticalAddonsOnly=true:NoSchedule` |
| **General workload pool** | Application pods, APIs, web servers | D4s_v5, autoscaler min 2 / max 20, no taint |
| **Memory-intensive pool** | Databases, caches, analytics | E-series, node label `workload=memory`, node affinity |
| **Spot pool** | Batch jobs, non-critical background work | D4s_v5 Spot, taint `kubernetes.azure.com/scalesetpriority=spot:NoSchedule` |
| **GPU pool** | ML training, inference, rendering | NC-series, taint `sku=gpu:NoSchedule`, NVIDIA device plugin |

**Node pool best practices:**
- Always separate system and user pools — system pods need guaranteed resources
- Use taints and tolerations to control pod placement (prevent non-GPU pods on GPU nodes)
- Enable cluster autoscaler on user pools with appropriate min/max bounds
- Consider ARM-based node pools (Dpsv5) for Linux workloads — best price-performance

## Networking Models

| Model | Pod IP Source | IP Consumption | Best For | Limitations |
|-------|-------------|---------------|----------|-------------|
| **kubenet** | Virtual pod CIDR (NAT to node) | Low (node IPs only from VNet) | Small clusters, IP-constrained networks | No direct pod-to-VNet communication, no NSG per pod |
| **Azure CNI** | VNet subnet (each pod gets VNet IP) | High (pre-allocates IPs per node) | Full VNet integration, NSG per pod | Requires large subnets, IP exhaustion risk |
| **Azure CNI Overlay** | Overlay CIDR (routed through node) | Low (overlay IPs, not VNet) | Large clusters, IP conservation | Recommended default for most new clusters |
| **Azure CNI + Cilium** | VNet or overlay + eBPF | Varies | Advanced network policies, observability | eBPF-based, requires Linux kernel support |

**Recommended default:** Azure CNI Overlay — balances IP conservation with VNet integration.

**Network policies:** Enforce pod-to-pod traffic rules. Options:
- **Calico** — mature, widely used, supports deny-all defaults
- **Cilium** — eBPF-based, better performance, L7 policies, advanced observability
- **Azure Network Policy** — Azure-native, simpler but less feature-rich

## Identity & Security

### Cluster Identity
- **Managed identity** for the cluster itself (control plane operations, node provisioning)
- **Kubelet identity** for node-level operations (pull images from ACR, access Key Vault)

### Pod Identity (Workload Identity)
- **Workload identity** (recommended) — federated identity credentials via OIDC
- Pod uses a Kubernetes service account annotated with Azure client ID
- Azure AD validates the K8s token via OIDC federation — no secrets stored in cluster
- Replaces deprecated AAD Pod Identity (aad-pod-identity)

### Secrets Management
- **Azure Key Vault CSI driver** — mount Key Vault secrets as volumes in pods
- Auto-rotation support (poll interval configurable)
- **external-secrets operator** — sync Key Vault to Kubernetes secrets
- Never store secrets in ConfigMaps or environment variables in plain text

### Security Layers
- **Microsoft Defender for Containers** — runtime threat detection, vulnerability scanning
- **Azure Policy for K8s (Gatekeeper)** — enforce policies (no privileged pods, required labels, resource limits)
- **Pod security admission** — enforce pod security standards (restricted, baseline, privileged)
- **Private cluster** — API server not publicly accessible (requires VPN, ExpressRoute, or private endpoint)
- **Image integrity** — verify container images with Notary v2 or Ratify

## Scaling

| Mechanism | Level | Trigger | Description |
|-----------|-------|---------|-------------|
| **HPA (Horizontal Pod Autoscaler)** | Pod | CPU, memory, custom metrics | Adjusts pod replica count based on metrics |
| **VPA (Vertical Pod Autoscaler)** | Pod | CPU, memory usage over time | Right-sizes pod resource requests (recommendation or auto mode) |
| **KEDA** | Pod | Event-driven (queues, streams, HTTP, cron) | Scales pods from 0 based on external event sources |
| **Cluster Autoscaler** | Node | Pending pods (insufficient resources) | Adds/removes nodes to fit workload. Per-node-pool config |
| **NAP (Node Auto Provisioning)** | Node | Pending pods + optimal VM selection | Karpenter-based, auto-selects best VM SKU for pending workloads |

**Best practice:** Combine HPA (or KEDA) for pods + Cluster Autoscaler for nodes. Set appropriate resource requests on all pods so the scheduler and autoscaler can make good decisions.

## Upgrade Strategy

### Version Upgrades (Quarterly)
- **Automatic channels:** None, Patch, Stable, Rapid, Node Image
- **Recommended:** Stable channel for production, Rapid for dev/test
- **Blue-green node pool:** Create new pool (new K8s version) → cordon old pool → migrate pods → drain and delete old pool
- **PDB (Pod Disruption Budget):** Always configure — prevents upgrades from terminating all pods of a service simultaneously

### Node Image Upgrades (Weekly)
- Separate from K8s version — OS patches, security fixes, runtime updates
- Configure automatic node image upgrades (SecurityPatch or NodeImage channel)
- Use maintenance windows to schedule during off-hours

### Surge Upgrades
- `max-surge` setting: extra nodes provisioned during upgrade (speed vs cost)
- `max-surge=1` — one extra node at a time (slower, cheaper)
- `max-surge=33%` — 33% extra nodes (faster, costs more during upgrade window)

## Observability

| Tool | What It Provides |
|------|-----------------|
| **Container Insights** | Node/pod metrics, logs, live data. Azure Monitor integration |
| **Prometheus + Grafana** | Azure Managed Prometheus + Managed Grafana. Community dashboards |
| **Azure Monitor alerts** | Alert on node health, pod restarts, OOM kills, resource pressure |
| **Log Analytics** | KQL queries over container logs. Cost-effective long-term retention |
| **Network Observability** | Hubble (Cilium) or Retina for pod-level network flow visibility |

## Common Mistakes

- Single node pool for everything — can't right-size, can't isolate system pods, can't use spot selectively
- No resource requests/limits on pods — scheduler can't bin-pack, autoscaler can't decide when to add nodes, OOM kills
- AKS without cluster autoscaler — paying for idle nodes 24/7 during off-peak
- Not setting PDBs — upgrades and node drains kill all replicas simultaneously
- Skipping network policies — all pods can talk to all pods by default (no microsegmentation)
- Using AAD Pod Identity instead of Workload Identity — deprecated, less secure
- Over-sized system node pool — 3x D8s_v5 for system pods that need 2 vCPU total
