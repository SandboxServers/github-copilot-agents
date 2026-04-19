# Compute Service Selection Matrix

## Decision Flow: Where Does This Workload Run?

```
Is it a web app or API?
├─ Yes → Does it need Kubernetes APIs or custom operators?
│        ├─ Yes → AKS
│        └─ No → Does it need microservice patterns (service discovery, Dapr, KEDA)?
│                 ├─ Yes → Container Apps
│                 └─ No → Is the team already on App Service?
│                          ├─ Yes → App Service (Web App for Containers or code deploy)
│                          └─ No → Container Apps (default modern choice)
├─ Event-driven / short-lived?
│  ├─ Yes → Azure Functions (Consumption for spiky, Premium for VNet/no cold start)
│  └─ Scheduled jobs? → Container Apps Jobs or Azure Functions Timer
├─ Batch / HPC?
│  └─ Azure Batch or AKS with GPU node pools
├─ Legacy / lift-and-shift / Windows services?
│  └─ Azure VMs (consider VMSS for scale)
└─ Static frontend?
   └─ Static Web Apps (with Functions backend if needed)
```

## Service Comparison Quick Reference

| Factor | App Service | Container Apps | AKS | Functions |
|--------|-------------|----------------|-----|-----------|
| Operational overhead | Low | Low-Medium | High | Low |
| Kubernetes API access | No | No | Yes | No |
| Scale to zero | No (always-on plan) | Yes | Yes (KEDA) | Yes (Consumption) |
| Custom domains/TLS | Built-in | Built-in | Manual/cert-manager | Built-in |
| VNet integration | Yes (Premium+) | Yes (native) | Yes (native) | Yes (Premium) |
| Deployment slots | Yes | Revisions + traffic split | Rolling/Blue-green | Slots (Premium) |
| GPU support | No | Yes (preview) | Yes | No |
| Windows containers | Yes | No | Yes | No |
| Max scale | 30 instances (standard) | 300 replicas | Node limits | 200 instances |
| Cold start concern | No | Yes (mitigate with min replicas) | No | Yes (Consumption) |

## SKU Selection Gotchas

**App Service:**
- Free/Shared: No custom domains with TLS, no VNet, no slots. Dev/test only.
- Basic: No autoscale, no slots. Tiny workloads.
- Standard: First tier with slots and autoscale. The starting point for production.
- Premium v3: VNet integration, better perf, zone redundancy. Production default.
- Isolated v2: Full network isolation (ASE). Compliance-driven only — expensive.
- **Gotcha**: Scaling is per App Service Plan, not per app. One hot app starves the others.
- **Gotcha**: Linux and Windows can't share a plan. Plan per OS.
- **Gotcha**: Deployment slots share the plan's resources. Slot = same compute, not extra.

**Container Apps:**
- Consumption: Pay per vCPU-second. Scale to zero. Default for most workloads.
- Dedicated (workload profiles): Predictable pricing, reserved compute. For steady-state.
- **Gotcha**: Consumption has a 4 vCPU / 8 GiB max per container. Bigger = Dedicated.
- **Gotcha**: No Windows container support. Windows workloads → App Service or AKS.
- **Gotcha**: Custom domains require manual DNS validation. No auto-cert like App Service.

**AKS:**
- System node pool: Always on, runs kube-system. Min 1-3 nodes.
- User node pools: Per-workload. Mix VM families. Spot pools for batch.
- **Gotcha**: AKS is "free" but you pay for VMs. An idle 3-node cluster still costs ~$300/mo.
- **Gotcha**: Version upgrades can break workloads. Test in staging. Use auto-upgrade channels carefully.
- **Gotcha**: CNI vs kubenet networking is hard to change later. Decide at cluster creation.

**Azure Functions:**
- Consumption: Scale to zero, pay per execution. Cold starts 1-10s.
- Flex Consumption: Improved cold starts, VNet, always-ready instances. New default.
- Premium (EP): No cold starts, VNet, larger instances. Steady throughput.
- Dedicated (App Service Plan): Use existing plan. No scale-to-zero.
- **Gotcha**: Consumption plan has 5-10 minute execution limit (default). 10 min max.
- **Gotcha**: Durable Functions state stored in Azure Storage — transaction costs add up at scale.
- **Gotcha**: Function app deployment replaces all functions atomically. No per-function deploy.
