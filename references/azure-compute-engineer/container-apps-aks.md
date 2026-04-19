## Container Apps vs AKS Decision Matrix

| Factor | Container Apps | AKS |
|--------|---------------|-----|
| **Kubernetes API access** | No | Full access |
| **Operational overhead** | Low (serverless) | High (cluster mgmt, upgrades, networking) |
| **Scale to zero** | Built-in | KEDA add-on required |
| **Custom operators / CRDs** | No | Yes |
| **Service mesh** | Envoy built-in | Istio, Linkerd, or any |
| **GPU workloads** | Limited (preview) | Full support (GPU node pools) |
| **Windows containers** | No | Yes |
| **Max replicas** | 1,000 per revision | 5,000 nodes per cluster |
| **Node pool control** | No (abstracted) | Full control (system/user pools, taints, labels) |
| **Dapr integration** | Built-in, managed | Manual installation |
| **Ingress** | Built-in (Envoy) | Ingress controller of choice (NGINX, AGIC, Contour) |
| **Pricing** | Per vCPU-second + GiB-second | Per node (VM cost) + cluster management fee |
| **Best for** | Microservices, APIs, background jobs, event-driven | Complex workloads, multi-team platforms, custom infra |

### When Container Apps Is Enough
- HTTP APIs and web apps (with or without Dapr)
- Background workers and event-driven processing
- Scale-to-zero workloads (cost optimization)
- Teams that don't want to manage Kubernetes
- Microservices with < 50 services

### When You Actually Need AKS
- Custom Kubernetes operators or CRDs
- Windows containers
- GPU node pools for ML training
- Service mesh requirements (Istio, Linkerd)
- Multi-tenant platforms with namespace isolation
- Existing Kubernetes expertise and tooling investment
- Compliance requiring dedicated control plane (private cluster)
