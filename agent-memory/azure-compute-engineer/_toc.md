# Azure Compute Engineer — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Compute Selection | [compute-selection.md](compute-selection.md) | When choosing between VMs, App Service, Container Apps, AKS, Functions, or Batch |
| VM Families | [vm-families.md](vm-families.md) | When selecting VM series — A/B/D/E/F/L/M/N, naming convention, B-series burstable |
| App Service | [app-service.md](app-service.md) | When working with App Service — tiers, deployment slots, scaling, networking |
| Container Apps vs AKS | [container-apps-aks.md](container-apps-aks.md) | When choosing between Container Apps and AKS or comparing features |
| Azure Functions | [azure-functions.md](azure-functions.md) | When working with Functions — hosting plans, cold start, Flex Consumption, triggers |
| Autoscaling | [autoscaling.md](autoscaling.md) | When designing autoscale strategies — VMSS, App Service, KEDA, HPA, cluster autoscaler |
| Cost Optimization | [cost-optimization.md](cost-optimization.md) | When optimizing compute costs — RIs, savings plans, spot VMs, right-sizing |
| AKS Deep Dive | [aks-deep-dive.md](aks-deep-dive.md) | When designing AKS clusters — node pools, networking models, upgrades, NAP |
| Batch & Anti-Patterns | [batch-anti-patterns.md](batch-anti-patterns.md) | When working with Azure Batch or reviewing compute designs for common mistakes |
| Anti-Patterns (Broad) | [anti-patterns.md](anti-patterns.md) | When reviewing compute architecture for sizing, availability, service selection, or cost mistakes |
| Gotchas | [gotchas.md](gotchas.md) | When encountering surprising behavior — VM resize, VMSS modes, AKS limits, spot eviction, cold starts |
| Best Practices | [best-practices.md](best-practices.md) | When designing production compute — right-sizing, availability zones, scaling, security, cost optimization |
| GPU & AKS Networking | [gpu-and-aks-networking.md](gpu-and-aks-networking.md) | When working with GPU/N-series for AI/ML or choosing AKS networking model (CNI Overlay vs flat vs kubenet) |
