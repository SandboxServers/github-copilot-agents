# Container Apps vs AKS

## Decision Matrix

Choose Container Apps for simplicity and serverless. Choose AKS for full Kubernetes control.

### When Container Apps Is Enough

- HTTP APIs and web apps (with or without Dapr sidecars)
- Background workers and event-driven processing with KEDA scalers
- Scale-to-zero workloads to minimize cost when idle
- Teams that don't want to manage Kubernetes infrastructure
- Microservices with fewer than ~50 services
- Revision-based traffic splitting for blue-green and canary deployments
- Jobs (scheduled or event-triggered batch processing)

### When You Need AKS

- Custom Kubernetes operators or CRDs (Custom Resource Definitions)
- Windows containers (Container Apps is Linux only)
- GPU node pools for ML training/inference
- Service mesh requirements (Istio, Linkerd, custom Envoy config)
- Multi-tenant platforms with namespace-level isolation and RBAC
- Existing Kubernetes expertise, tooling, and Helm charts
- Compliance requiring dedicated control plane (private cluster with private API server)
- Advanced networking: custom CNI plugins, multiple ingress controllers, network policies
- Very large scale (> 1,000 replicas per service or > 5,000 total pods)

## Feature Comparison

| Feature | Container Apps | AKS |
|---------|---------------|-----|
| **Kubernetes API access** | No (abstracted) | Full K8s API |
| **Operational overhead** | Low (serverless/managed) | High (cluster upgrades, node management, networking) |
| **Scale to zero** | Built-in (Consumption plan) | KEDA add-on required (nodes still cost money) |
| **Autoscaling** | HTTP concurrent requests, KEDA scalers, CPU/memory | HPA, VPA, Cluster Autoscaler, KEDA, NAP |
| **Custom operators / CRDs** | No | Yes (full K8s extensibility) |
| **Service mesh** | Envoy proxy built-in (managed) | Istio, Linkerd, or any (self-managed) |
| **Dapr integration** | Built-in, fully managed | Manual install and management |
| **GPU workloads** | Limited (preview, dedicated plan) | Full support (GPU node pools, NVIDIA device plugin) |
| **Windows containers** | Not supported | Supported (Windows node pools) |
| **Max replicas** | 1,000 per revision | 5,000 nodes per cluster, 150,000 pods |
| **Node pool control** | No (abstracted in Consumption, workload profiles in Dedicated) | Full (system/user pools, taints, labels, VM size per pool) |
| **Ingress** | Built-in Envoy-based with automatic TLS | Choice of ingress controller (NGINX, AGIC, Contour, Traefik) |
| **Networking** | Environment-level VNet integration | Full CNI (Azure CNI, CNI Overlay, Cilium) |
| **Network policies** | Not supported | Calico, Cilium, Azure Network Policy |
| **Secrets management** | Managed secrets, Key Vault references | K8s secrets, Key Vault CSI driver, external-secrets operator |
| **Observability** | Built-in metrics, Azure Monitor, Log Analytics | Prometheus, Grafana, Azure Monitor, Container Insights |
| **Pricing model** | Per vCPU-second + per GiB-second (Consumption) | Per node VM cost + optional management fee |
| **Cost when idle** | $0 (Consumption, scale-to-zero) | Node VMs always running (min ~$100/mo for small cluster) |

## Container Apps Architecture

```
Container Apps Environment (VNet boundary)
├─ App 1 (Revision 1 — 80% traffic)
│  ├─ Replica 1
│  ├─ Replica 2
│  └─ Replica 3
├─ App 1 (Revision 2 — 20% traffic, canary)
│  └─ Replica 1
├─ App 2 (background worker, KEDA scaler on Service Bus)
│  └─ Replicas 0-10 (auto)
├─ Job 1 (scheduled, cron-triggered)
└─ Dapr sidecars (optional, per-app)
```

**Key concepts:**
- **Environment**: Shared boundary for apps. Provides VNet, Log Analytics, Dapr components
- **Revision**: Immutable snapshot of app config + container image. Traffic splitting between revisions
- **Replica**: Running instance of a revision. Scaled by rules (HTTP, KEDA, CPU/memory)
- **Job**: Triggered or scheduled container execution (not long-running)

## AKS Architecture (Summary)

```
AKS Cluster (Managed control plane — free)
├─ System Node Pool (CoreDNS, kube-proxy, metrics-server)
│  └─ 3x Standard_D2s_v5 (tainted: CriticalAddonsOnly)
├─ General User Node Pool (application workloads)
│  └─ 2-20x Standard_D4s_v5 (cluster autoscaler)
├─ Spot Node Pool (batch jobs, non-critical)
│  └─ 0-10x Standard_D4s_v5 Spot (tainted: spot priority)
└─ GPU Node Pool (ML inference)
   └─ 0-4x Standard_NC4as_T4_v3 (tainted: GPU workloads)
```

## Migration: Container Apps → AKS

If you outgrow Container Apps, migration to AKS is straightforward:

1. Container images are the same — no rebuild needed
2. Convert Container Apps YAML to Kubernetes Deployments/Services
3. Map Container Apps ingress to Kubernetes Ingress or Gateway API
4. Replace managed Dapr with self-installed Dapr on AKS
5. Replace KEDA scaling rules with KEDA ScaledObjects (same scalers, different config format)
6. Set up AKS networking (Azure CNI Overlay recommended)
7. Configure cluster autoscaler, HPA, and monitoring

## Cost Comparison (Example: 2 microservices)

| Scenario | Container Apps (Consumption) | AKS (Minimum) |
|----------|----------------------------|---------------|
| Idle (no traffic) | ~$0/month | ~$150/month (3 system nodes) |
| Light traffic (100 req/sec) | ~$20-50/month | ~$200/month (3 system + 2 user nodes) |
| Moderate traffic (1,000 req/sec) | ~$100-300/month | ~$400/month (3 system + 4 user nodes) |
| Heavy traffic (10,000 req/sec) | ~$500-1,500/month | ~$800/month (3 system + 8 user nodes) |

**Crossover point:** AKS becomes more cost-effective at sustained high scale (~5,000+ req/sec) or when running many services that fill nodes efficiently.

## Common Mistakes

- Starting with AKS for 2-3 services when Container Apps would suffice — massive overhead for small workloads
- Using Container Apps Dedicated plan when Consumption handles the load — unnecessary fixed cost
- Not setting resource limits in Container Apps — replicas can consume more than expected
- Running AKS without cluster autoscaler — paying for idle nodes 24/7
- Choosing AKS because "we might need it later" — migration from Container Apps is well-documented
- Ignoring Container Apps Jobs for batch workloads — simpler than running Jobs on AKS
