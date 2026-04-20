# GPU Compute & AKS Networking Patterns

Two significant gaps in the existing knowledge base: GPU/N-series compute patterns for AI/ML workloads, and AKS networking model selection.

## GPU / N-Series Compute Patterns

### When to Use GPU VMs

| Workload | Recommended SKU Family | Notes |
|----------|----------------------|-------|
| **ML model training (deep learning)** | NC-series (T4), ND-series (A100, H100) | Data parallelism across multiple GPUs; use MIG for multi-instance |
| **ML inference (real-time)** | NC-series (T4), NV-series (A10) | Lower GPU count sufficient; optimize for latency over throughput |
| **Rendering / visualization** | NV-series | Optimized for graphics workloads; AMD GPUs available |
| **HPC / simulation** | ND-series with InfiniBand | RDMA-capable for MPI workloads; cross-node GPU communication |
| **LLM serving (vLLM, Triton)** | ND-series (A100 80GB, H100) | Large VRAM required; use tensor parallelism across GPUs |

### GPU on AKS

- **Use dedicated GPU node pools:** Separate from CPU workloads — prevents GPU idle waste and scheduling conflicts
- **Enable cluster autoscaler with min=0 on GPU pools:** GPU VMs are expensive ($1-10+/hr) — scale to zero when no GPU workloads are queued
- **Use NVIDIA device plugin (auto-installed by AKS):** Exposes `nvidia.com/gpu` resource for pod scheduling
- **Use spot GPU VMs for training workloads:** Training is fault-tolerant with checkpointing — up to 90% savings
- **Enable multi-instance GPU (MIG) on A100/H100:** Partition one GPU into up to 7 instances for smaller inference workloads
- **Use node taints on GPU pools:** `sku=gpu:NoSchedule` prevents non-GPU workloads from landing on expensive nodes
- **Use tolerations on GPU pods:** Match the taint so only GPU workloads schedule on GPU nodes

### GPU Cost Optimization

| Strategy | Savings | Trade-off |
|----------|---------|-----------|
| **Spot GPU VMs** | Up to 90% | Eviction risk — implement checkpointing |
| **Scale-to-zero GPU node pools** | Eliminates idle cost | 5-10 min cold start for new GPU nodes |
| **MIG partitioning** | Better utilization per GPU | Only on A100/H100; adds scheduling complexity |
| **Right-size GPU SKU** | Avoid paying for unused VRAM | Smaller GPUs may not fit large models |
| **Reserved instances for stable inference** | 30-60% | Locked to size family and region |

### AI Toolchain Operator (KAITO)

AKS includes the **AI toolchain operator** for simplified model deployment:
- Pre-built model images for Llama, Falcon, Phi, Mistral
- Automatic GPU node provisioning and model serving
- Supports fine-tuning, tool calling, and MCP server integration
- Reduces setup from days to minutes for common LLM serving scenarios

## AKS Networking Models

### Decision Matrix

| Model | IP Source for Pods | VNet IP Usage | Max Scale | Direct Pod Access from VNet | Best For |
|-------|-------------------|---------------|-----------|---------------------------|----------|
| **Azure CNI Overlay** | Private CIDR (overlay) | Conserves IPs | Highest (1000 nodes, 250 pods/node) | No (SNAT through node) | Most clusters — recommended default |
| **Azure CNI (Node Subnet)** | VNet subnet | High consumption | Limited by subnet size | Yes | Legacy; direct pod access needed |
| **Azure CNI (Pod Subnet)** | Dedicated pod subnet | Moderate | Large | Yes | Direct pod access with better IP management |
| **Kubenet** | Private CIDR (UDR-based) | Conserves IPs | 400 nodes max | No | Small clusters; simple requirements |

### When to Use Each Model

**Azure CNI Overlay (recommended for most):**
- Best IP conservation — pods use separate overlay CIDR, nodes use VNet IPs
- Maximum cluster scale support
- Simple configuration
- Trade-off: pods not directly reachable from VNet (use Services/Ingress)

**Azure CNI (flat network):**
- Required when external systems must reach pods by IP directly
- Required for some advanced features (certain CSI drivers, specific compliance)
- Trade-off: one VNet IP per pod — plan for IP exhaustion

**Kubenet:**
- Simplest model, lowest IP usage
- Limited to 400 nodes
- Requires manual UDR management
- Trade-off: no support for Azure network policies, Windows nodes, or virtual nodes

### Networking Gotchas

- **CNI choice is immutable after cluster creation** — you cannot switch networking models on an existing cluster
- **Overlay CIDR must not overlap with VNet or service CIDR** — plan address spaces before cluster creation
- **Azure CNI (flat): IP exhaustion is the #1 scaling blocker** — /24 subnet = max ~250 pods total, not per node
- **Network policies:** Azure policy engine only works with Azure CNI; Calico works with both CNI and Kubenet (Linux only)
- **Dual-stack (IPv4+IPv6):** Supported on CNI Overlay; requires Kubernetes 1.26.3+

## References

- [GPU best practices for AKS](https://learn.microsoft.com/azure/aks/best-practices-gpu)
- [AKS GPU workloads architecture](https://learn.microsoft.com/azure/architecture/reference-architectures/containers/aks-gpu/gpu-aks)
- [AKS CNI networking overview](https://learn.microsoft.com/azure/aks/concepts-network-cni-overview)
- [Azure CNI Overlay overview](https://learn.microsoft.com/azure/aks/concepts-network-azure-cni-overlay)
- [Network topology for AKS (CAF)](https://learn.microsoft.com/azure/cloud-adoption-framework/scenarios/app-platform/aks/network-topology-and-connectivity)
