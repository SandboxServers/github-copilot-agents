# Service Affinity Matrix — What Plays Well Together

| Primary Service | Natural Companions | Why |
|----------------|-------------------|-----|
| App Service | Key Vault, SQL Database, Redis Cache, Front Door, App Insights | Classic PaaS stack. Well-integrated, minimal config. |
| Container Apps | Cosmos DB, Service Bus, Dapr, KEDA, Azure Monitor | Cloud-native microservices. Event-driven scaling. |
| AKS | ACR, Key Vault CSI, Azure CNI, Prometheus/Grafana, Flux/ArgoCD | Full Kubernetes ecosystem. Maximum flexibility. |
| Functions | Event Grid, Service Bus, Storage Queues, Cosmos DB, API Management | Event-driven glue. Connects services together. |
| Static Web Apps | Functions backend, Cosmos DB, Entra auth, CDN | JAMstack. Global distribution, low cost. |
