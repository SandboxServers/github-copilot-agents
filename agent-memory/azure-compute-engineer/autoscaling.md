# Autoscaling Patterns

## The Autoscaling Stack

Every Azure compute service scales differently. Understand which layer applies to your workload.

```
Layer 1: Infrastructure / Node Scale (VMs behind the scenes)
├─ VMSS Autoscale — scale VM count based on metrics or schedule
├─ AKS Cluster Autoscaler — add/remove nodes when pods are pending
├─ AKS Node Auto Provisioning (NAP) — auto-select optimal VM SKU for pending workloads
└─ VMSS Predictive Autoscale — ML-based pre-scaling for cyclical patterns

Layer 2: Application / Container Scale (instances/replicas)
├─ App Service Autoscale (rule-based) — CPU, memory, queue length, schedule, custom metrics
├─ App Service Automatic Scaling — HTTP traffic-based with pre-warmed instances
├─ Container Apps Scaling Rules — HTTP concurrency, KEDA scalers, CPU/memory
├─ AKS HPA — CPU/memory-based pod replica scaling
├─ AKS VPA — right-size pod resource requests automatically
└─ KEDA — event-driven pod scaling (queue length, stream lag, HTTP, cron, custom)

Layer 3: Serverless Scale (fully managed)
├─ Functions Consumption — automatic per-invocation, scale-to-zero
├─ Functions Flex Consumption — per-instance with concurrency, always-ready option
└─ Functions Premium — pre-warmed, no cold start, event-driven scale-out
```

## VMSS Autoscaling

| Rule Type | Trigger | Use Case |
|-----------|---------|----------|
| **Metric-based** | CPU > 70%, memory > 80%, queue length > 100 | Variable traffic reacting to real-time load |
| **Schedule-based** | Scale to 10 at 8 AM, scale to 3 at 8 PM | Predictable traffic patterns (business hours) |
| **Predictive** | ML model from historical metrics | Cyclical patterns where reactive scaling is too slow |
| **Combined** | Metric + schedule together | Schedule for baseline, metric for spikes above baseline |

**VMSS scaling rules:** Scale-out and scale-in are separate rules. Always configure both. Use different thresholds (e.g., scale out at CPU > 75%, scale in at CPU < 25%) to avoid flapping.

## App Service Scaling

| Method | Tier | Trigger | Scope |
|--------|------|---------|-------|
| **Manual** | Basic+ | Fixed count | App Service Plan |
| **Rule-based autoscale** | Standard+ | CPU, memory, HTTP queue, custom metrics, schedule | App Service Plan (all apps scale together) |
| **Automatic scaling** | Premium v3+ | HTTP traffic (managed by Azure) | Per-app (independent scaling) |

**Automatic scaling** (Premium v3) features:
- Always-ready instances: minimum warm instances per app
- Pre-warmed instances: buffer beyond always-ready for cold start mitigation
- Maximum burst limit: cap per app to prevent runaway scaling
- No manual metric rules needed — Azure manages based on HTTP load

## Container Apps Scaling

KEDA-powered scaling with built-in support for common scalers:

| Scaler | Trigger Metric | Use Case |
|--------|---------------|----------|
| **HTTP** | Concurrent requests per replica | Web APIs, HTTP services |
| **Azure Service Bus** | Queue/topic message count | Message-driven processing |
| **Azure Storage Queue** | Queue length | Async job processing |
| **Azure Event Hubs** | Consumer group lag | Stream processing |
| **Cron** | Schedule expression | Periodic batch jobs |
| **PostgreSQL / MySQL** | Query result value | Database-driven scaling |
| **TCP** | Active connections | TCP-based services |
| **CPU** | CPU utilization percentage | General compute scaling |
| **Memory** | Memory utilization percentage | Memory-intensive workloads |
| **Custom** | Any KEDA-supported external scaler | Prometheus, Datadog, custom metrics |

**Scale-to-zero:** Consumption plan scales to 0 replicas when no events/requests. First request triggers cold start (container pull + startup). Set min replicas > 0 to avoid cold starts at the cost of idle charges.

## AKS Scaling

| Mechanism | Level | What It Does | Key Settings |
|-----------|-------|-------------|-------------|
| **HPA** | Pod replicas | Scales pods based on CPU, memory, or custom metrics | `minReplicas`, `maxReplicas`, `targetCPUUtilization` |
| **VPA** | Pod resources | Adjusts pod CPU/memory requests based on actual usage | `updateMode` (Off=recommend, Auto=apply) |
| **KEDA** | Pod replicas | Event-driven scaling (queues, streams, HTTP) | ScaledObject with trigger config, `minReplicaCount=0` |
| **Cluster Autoscaler** | Nodes | Adds nodes when pods are pending, removes when underutilized | `min-count`, `max-count`, `scale-down-delay-after-add` |
| **NAP** | Nodes | Auto-selects optimal VM SKU for pending workloads | Karpenter-based, NodePool CRD with constraints |

**Combining HPA + Cluster Autoscaler:** HPA creates more pods → pods pending (no room) → Cluster Autoscaler adds nodes → pods scheduled. Scale-in reverses: HPA reduces pods → nodes underutilized → Cluster Autoscaler removes nodes.

## Functions Scaling

| Plan | Scale Model | Concurrency | Scale-to-Zero |
|------|------------|-------------|---------------|
| **Consumption** | Per-invocation, one instance per trigger event (batching for queues) | Single invocation per instance (most triggers) | Yes |
| **Flex Consumption** | Per-instance with configurable concurrency | Multiple concurrent invocations per instance | Yes (with always-ready option) |
| **Premium** | Pre-warmed + dynamic scale-out | Configurable per trigger type | No (min 1 instance) |

## Scale-to-Zero Capabilities

| Service | Scale to Zero? | Cold Start | Idle Cost | Resume Trigger |
|---------|---------------|-----------|-----------|----------------|
| **Functions Consumption** | Yes | 1-10 seconds | $0 | Event (HTTP, queue, timer, etc.) |
| **Functions Flex Consumption** | Yes (optional always-ready) | Sub-second if always-ready | $0 (or always-ready cost) | Event |
| **Container Apps (Consumption)** | Yes | Seconds (image pull + start) | $0 | HTTP request or KEDA event |
| **AKS + KEDA** | Pods: yes. Nodes: no (cluster autoscaler has floor) | Seconds (pod) + minutes (node scale-up) | Node VM cost remains | KEDA event |
| **App Service** | No (min 1 instance per plan) | N/A | Plan cost | N/A |
| **VMSS** | Can scale to 0 instances | Minutes (VM provision) | $0 (but lose all state) | Metric or schedule rule |

## Best Practices

- **Always set both min AND max** — unbounded scaling causes cost surprises or resource exhaustion
- **Test scale-out AND scale-in** — scale-in bugs are common (connections draining, sticky sessions)
- **Set cooldown periods** — prevent rapid scale-out/scale-in oscillation (flapping). Default 5 min is often too short
- **Use leading indicators** — queue length and request rate predict load before CPU spikes
- **Don't scale on lagging metrics** — average CPU over 10 minutes is too slow for burst traffic
- **Monitor scaling events** — Azure Monitor activity log shows every scale action with reason and result
- **Pre-scale for known events** — schedule-based scaling before a product launch or marketing campaign
- **Resource requests matter for AKS** — HPA and Cluster Autoscaler depend on accurate resource requests to make decisions
- **Avoid VPA + HPA on the same metric** — both adjusting CPU causes conflicts. Use VPA for memory, HPA for CPU
