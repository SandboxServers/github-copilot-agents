## Autoscaling at Every Layer

### The Autoscaling Stack

```
Layer 1: VM / Infrastructure Scale
├─ VMSS Autoscale — scale VM count based on metrics (CPU, memory, custom)
├─ AKS Cluster Autoscaler — add/remove nodes based on pending pods
├─ AKS Node Autoprovision (NAP/Karpenter) — auto-select optimal VM SKU for pending workloads
└─ Predictive Autoscale — ML-based pre-scaling for VMSS with cyclical patterns

Layer 2: Application / Container Scale
├─ App Service Autoscale — rule-based (CPU, memory, queue, schedule)
├─ App Service Automatic Scaling — traffic-based with prewarmed instances
├─ Container Apps Scaling Rules — HTTP concurrent requests, KEDA scalers, CPU/memory
├─ AKS Horizontal Pod Autoscaler (HPA) — CPU/memory-based pod scaling
├─ KEDA — event-driven pod scaling (queue length, HTTP, custom metrics)
└─ Vertical Pod Autoscaler (VPA) — right-size pod CPU/memory requests

Layer 3: Serverless Scale
├─ Functions Consumption — automatic, event-driven, scale-to-zero
├─ Functions Flex Consumption — same but with always-ready and VNet
└─ Functions Premium — pre-warmed, no cold start, VNet
```

### KEDA Scalers (Key Ones)

| Scaler | Trigger | Use Case |
|--------|---------|----------|
| Azure Service Bus | Queue/topic message count | Message processing |
| Azure Storage Queue | Queue length | Async job processing |
| HTTP | Concurrent requests | Web APIs |
| Cron | Schedule | Periodic jobs |
| PostgreSQL / MySQL | Query result | DB-driven scaling |
| Azure Event Hubs | Consumer group lag | Stream processing |
| Prometheus | Custom metric | App-specific scaling |

### Scale-to-Zero Capabilities

| Service | Scale to Zero? | Warm-Up Time | Cost When Idle |
|---------|---------------|-------------|----------------|
| Functions Consumption | Yes | 1-10 seconds (cold start) | $0 |
| Functions Flex Consumption | Yes (with always-ready option) | Sub-second if always-ready | $0 (or always-ready cost) |
| Container Apps | Yes | Seconds (container pull + start) | $0 |
| AKS + KEDA | Yes (pods only, nodes still running) | Seconds (pod) + minutes (node) | Node cost still applies |
| App Service | No (min 1 instance per plan) | N/A | Plan cost |
| VMs/VMSS | VMSS can scale to 0 | Minutes | $0 (but lose state) |
