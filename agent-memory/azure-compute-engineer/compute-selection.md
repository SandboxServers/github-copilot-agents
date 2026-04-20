# Compute Selection

## Quick Decision Framework

Start here — match your workload pattern to the right Azure compute service:

- **Need scale-to-zero?** → Azure Functions (Consumption) or Container Apps (Consumption)
- **Simple web app or API?** → App Service (fastest path for .NET, Node, Python, Java)
- **Containerized microservices?** → Container Apps (simple) or AKS (complex/multi-team)
- **Full OS control or custom software?** → Virtual Machines (or VMSS for scale-out)
- **Batch/HPC workloads?** → Azure Batch or Spot VMs in VMSS
- **GPU workloads (AI/ML, rendering)?** → AKS with GPU node pools or N-series VMs
- **Windows containers?** → AKS (Container Apps does not support Windows containers)
- **Static site with serverless API?** → Static Web Apps
- **Long-running workflows?** → Durable Functions or Container Apps Jobs
- **Don't know yet?** → Container Apps covers the widest sweet spot with lowest lock-in

## Detailed Comparison

| Service | Min Monthly Cost | Max Scale | Cold Start | VNet Support | Windows Support | GPU Support |
|---------|-----------------|-----------|------------|--------------|-----------------|-------------|
| **Azure Functions (Consumption)** | $0 (scale-to-zero) | 200 instances | Yes (1-10s) | No (outbound only) | Yes | No |
| **Azure Functions (Flex Consumption)** | $0 (scale-to-zero) | 1,000 instances | Reduced (always-ready option) | Yes (full VNet) | No (Linux only) | No |
| **Azure Functions (Premium)** | ~$155 (EP1, 1 instance) | 100 instances | No (pre-warmed) | Yes (full VNet + private endpoints) | Yes | No |
| **App Service (Basic B1)** | ~$55 | 3 instances | No | No | Yes | No |
| **App Service (Standard S1)** | ~$73 | 10 instances | No | Yes (regional) | Yes | No |
| **App Service (Premium v3 P1v3)** | ~$110 | 30 instances | No | Yes (regional + private endpoints) | Yes | No |
| **App Service (Isolated v2 I1v2)** | ~$298 | 100 instances | No | Yes (ASE, dedicated VNet) | Yes | No |
| **Container Apps (Consumption)** | $0 (scale-to-zero) | 1,000 replicas | Yes (container pull) | Yes (environment VNet) | No | No (preview) |
| **Container Apps (Dedicated)** | ~$75 (D4 workload profile) | 1,000 replicas | No | Yes | No | Yes (preview) |
| **AKS** | ~$0 control plane + node VMs | 5,000 nodes/cluster | No (pods on running nodes) | Yes (full CNI integration) | Yes | Yes (GPU node pools) |
| **VMs / VMSS** | ~$15 (B1s) | 1,000 per VMSS | No | Yes (VNet-native) | Yes | Yes (N-series) |
| **Azure Batch** | $0 (pool VMs only) | 10,000 cores (default quota) | Pool creation time | Yes | Yes | Yes (N-series) |

## Decision Tree

```
What are you running?
├─ Legacy app / lift-and-shift / Windows services / custom OS?
│  └─ Virtual Machines (or VMSS for horizontal scale)
├─ Web app or API (HTTP-based)?
│  ├─ Simple web app, team knows PaaS? → App Service
│  ├─ Containerized microservices, moderate complexity? → Container Apps
│  ├─ Full Kubernetes ecosystem, operators, service mesh? → AKS
│  └─ Static site + API? → Static Web Apps
├─ Event-driven / trigger-based?
│  ├─ Short-lived stateless functions? → Azure Functions (Consumption or Flex)
│  ├─ Event-driven containers, scale-to-zero? → Container Apps (KEDA scalers)
│  └─ Durable workflows with orchestration? → Durable Functions
├─ Batch / HPC / embarrassingly parallel?
│  └─ Azure Batch (or VMSS with Spot for cost-sensitive)
├─ GPU workloads (AI/ML training, rendering)?
│  └─ AKS with GPU node pools or N-series VMs
└─ Don't know yet?
   └─ Container Apps — lowest operational overhead, broadest coverage
```

## Key Trade-offs

### Operational Overhead (low → high)
Functions → Static Web Apps → App Service → Container Apps → Azure Batch → AKS → VMs/VMSS

### Flexibility (low → high)
Functions → App Service → Container Apps → AKS → VMs/VMSS

### Cost at Low Scale (low → high)
Functions Consumption ($0) → Container Apps Consumption ($0) → App Service Basic (~$55) → AKS (node cost) → VMs

### Cost at High Scale (per-unit, low → high)
Reserved VMs → AKS (right-sized nodes) → App Service Premium → Container Apps Dedicated → Functions Premium

## Common Mistakes

- Choosing AKS for 2-3 microservices — Container Apps is simpler and cheaper
- Using VMs for web apps that could run on App Service — unnecessary OS management burden
- Functions Consumption for latency-sensitive APIs — cold starts will ruin P99 latency
- App Service Isolated for VNet needs — Premium v3 with regional VNet integration is cheaper
- Skipping Container Apps because "we might need AKS later" — migration path is straightforward
- Over-provisioning App Service plans for single small apps — pack multiple apps per plan

## Migration Paths

| From | To | Difficulty | Notes |
|------|----|-----------|-------|
| App Service → Container Apps | Low-Medium | Containerize app, map config to Container Apps environment variables |
| Container Apps → AKS | Medium | Export container specs, create Kubernetes manifests, configure networking |
| VMs → App Service | Medium-High | Refactor to remove OS dependencies, adopt PaaS patterns |
| VMs → AKS | High | Containerize, create K8s manifests, rethink networking and storage |
| Functions → Container Apps | Low | Package function as container, use KEDA HTTP scaler |
| AKS → Container Apps | Low-Medium | Simplify manifests, map ingress to Container Apps built-in routing |
