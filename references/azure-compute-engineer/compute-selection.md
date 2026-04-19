## Compute Platform Decision Tree

```
What are you running?
├─ Legacy app / lift-and-shift / Windows services / custom OS?
│  └─ Virtual Machines (or VMSS for scale)
├─ Web app or API (HTTP-based)?
│  ├─ Simple web app, team knows .NET/Node/Python/Java? → App Service
│  ├─ Containerized microservices, moderate complexity? → Container Apps
│  ├─ Full Kubernetes ecosystem, custom operators, service mesh? → AKS
│  └─ Static site + API? → Static Web Apps
├─ Event-driven / trigger-based?
│  ├─ Short-lived, stateless functions? → Azure Functions (Consumption or Flex)
│  ├─ Event-driven containers, scale-to-zero? → Container Apps (KEDA)
│  └─ Durable workflows? → Durable Functions or Logic Apps
├─ Batch / HPC / embarrassingly parallel?
│  └─ Azure Batch (or AKS with GPU node pools for ML)
├─ GPU workloads (AI/ML training, rendering)?
│  └─ VMs (N-series) or AKS with GPU node pools
└─ Don't know yet?
   └─ Start with Container Apps — it covers the widest sweet spot
```
