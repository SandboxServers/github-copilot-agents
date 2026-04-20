# Compute Selection

Decision matrix for Azure compute:

| Service | Best For | Scaling | Min Monthly | Complexity |
|---------|----------|---------|-------------|------------|
| App Service | Web apps, APIs | Manual/auto, up to 30 instances (Standard), 100 (Premium) | ~$55 (B1) | Low |
| Container Apps | Microservices, event-driven containers | KEDA-based, scale to zero | $0 (consumption) | Medium |
| AKS | Complex microservices, teams with K8s expertise | HPA, cluster autoscaler, KEDA, NAP | ~$73 (B4ms node) | High |
| Functions | Event processing, integrations | Per-execution, scale to zero | $0 (consumption) | Low |
| VMs/VMSS | Legacy apps, full OS control, lift-and-shift | VMSS autoscale | ~$15 (B1s) | Medium |

## App Service

**SKU Tiers:**
- **Free/Shared**: Dev only. No custom TLS, no VNet, no slots. 60/240 CPU min/day limits.
- **Basic (B1-B3)**: No autoscale, no slots. Small production workloads with predictable traffic.
- **Standard (S1-S3)**: Autoscale up to 30 instances. 5 deployment slots. Backups. Custom domains + managed certs.
- **Premium v3 (P0v3-P3mv3)**: Zone redundancy, up to 100 instances, enhanced performance. VNet integration.
- **Isolated v2 (I1v2-I6v2)**: App Service Environment. Full network isolation. Compliance/regulatory requirements only.

**When to choose:** Web apps, APIs, line-of-business apps. Teams wanting simplicity. Deployment slots for staging. Custom domains + managed certs. VNet integration for outbound.

**When NOT to choose:** Microservices that need independent scaling (scaling is per plan, not per app). Workloads needing scale-to-zero. Apps needing full OS control.

**Key limits:** Linux and Windows apps cannot share a plan. Slots consume same plan resources (not additional compute). One runaway app starves siblings on same plan.

## Container Apps

**Pricing models:**
- **Consumption**: Pay per vCPU-second and GiB-second. Scale to zero. Default for variable workloads.
- **Workload profiles (Dedicated)**: Reserved compute with predictable pricing. D4, D8, D16, D32 profiles and GPU options.

**When to choose:** Microservices without K8s expertise. Event-driven architectures. Workloads that benefit from scale-to-zero. Dapr for service-to-service communication. Built-in HTTPS ingress. Revision-based traffic splitting.

**When NOT to choose:** Need full Kubernetes API. Need Windows containers. Need more than 4 vCPU / 8 GiB per container on consumption. Custom operators or service meshes.

**Key limits:** Consumption profile maxes at 4 vCPU / 8 GiB per container. No Windows container support. Max 100 revisions per app. Environment = shared boundary for networking and logging.

## AKS

**Architecture:** System + user node pools. Multiple VM families (CPU, GPU, memory-optimized). Azure CNI Overlay recommended for new clusters. Managed identity for cluster. Workload identity for pods.

**When to choose:** Teams with Kubernetes expertise. Complex workloads needing custom operators, service meshes (Istio, Linkerd), GPU scheduling, or multi-tenant isolation. Need full K8s API access.

**When NOT to choose:** Small teams without K8s experience. Simple web apps (use App Service). Microservices that don't need K8s-specific features (use Container Apps). Cost-sensitive workloads (idle cluster ~$300/month).

**Key limits:** CNI choice is immutable after creation. Node pool VM SKU cannot be changed (create new pool and migrate). K8s version upgrades can break workloads. Azure Key Vault CSI driver for secrets. Microsoft Defender for Containers for security.

## Functions

**Pricing plans:**
- **Consumption**: Scale to zero. Pay per execution (first 1M free). 5-min default / 10-min max timeout. Cold starts 1-10 seconds.
- **Flex Consumption**: VNet support. Fast scaling with always-ready instances. Per-function scaling. Recommended new default.
- **Premium (EP1-EP3)**: Pre-warmed instances (no cold starts). VNet integration. No timeout limit. Steady throughput workloads.
- **Dedicated (App Service Plan)**: Use existing plan compute. No scale-to-zero. Legacy option.

**When to choose:** Event processing, integrations, scheduled jobs, lightweight APIs, glue logic. Durable Functions for orchestrations (fan-out/fan-in, human interaction, chaining).

**When NOT to choose:** Long-running processes on Consumption plan. Workloads needing persistent connections (WebSockets). Complex APIs better served by App Service or Container Apps.

**Key limits:** Consumption execution time limits are strict. Deployment replaces all functions atomically. Durable Functions storage transactions add up at scale.

## VMs / VMSS

**When to choose:** Legacy applications that cannot be containerized. Lift-and-shift migrations. Workloads requiring specific OS configurations, kernel modules, or GPU drivers. Full OS control needed.

**When NOT to choose:** New applications — always evaluate PaaS first. App Service, Container Apps, or Functions can almost always do it better with less operational burden.

**Key limits:** You manage the OS, patching, security, networking. VMSS for horizontal scaling with autoscale rules. Availability Zones for HA. Spot VMs for fault-tolerant batch processing.
