## Domain Interconnections

### How Specialties Affect Each Other

| Decision In | Ripples To | Why It Matters |
|-------------|-----------|----------------|
| **Network topology** | Everything | VNet structure, private endpoints, DNS zones, and NSGs constrain what compute can reach what data. Get networking wrong and everything else is a workaround. |
| **Compute selection** | Storage, networking, cost | App Service vs AKS vs Container Apps vs Functions each have different VNet integration models, storage mount options, and cost profiles. Compute choice dictates storage access patterns. |
| **Database selection** | Compute, integration, cost | Cosmos DB vs SQL vs PostgreSQL each have different connection models, consistency guarantees, and pricing. Database choice constrains integration patterns and affects compute scaling. |
| **Storage tier** | Compute, data, cost | Hot vs cool vs archive, ADLS Gen2 vs Blob vs Files — each has different access latency and cost. Storage choices affect data pipeline performance and compute I/O patterns. |
| **Integration pattern** | Reliability, monitoring, compute | Synchronous vs asynchronous, point-to-point vs pub/sub — integration patterns determine failure modes, observability requirements, and compute scaling behavior. |
| **AI/ML workload** | Compute, storage, networking, cost | GPU compute has different availability zones, networking constraints, and cost. Training data pipelines require storage and network bandwidth. Inferencing endpoints need dedicated compute. |
| **Monitoring design** | All domains | What you can observe determines what you can troubleshoot. Monitoring must be wired into every domain from day one, not added after production issues emerge. |
| **Identity model** | All domains | Managed identity, RBAC, and network identity (private endpoints, service endpoints) form the security foundation. Identity mistakes are the hardest to fix after deployment. |

### Cross-Cutting Concerns

Three concerns span every specialty and cannot be delegated to a single specialist:

1. **Networking**: Every service needs network connectivity. Private endpoints, VNet integration, DNS resolution, NSGs, and routing affect every domain. The network engineer provides the foundation, but every specialist must understand how their domain uses it.

2. **Identity**: Managed identities, RBAC assignments, Key Vault references, and Entra authentication affect every service. The identity model must be consistent across all domains — inconsistency creates security gaps and operational confusion.

3. **Monitoring**: Application Insights, Log Analytics, diagnostic settings, alerts, and dashboards must cover every domain. Each specialist adds their domain-specific telemetry, but the observability platform must be unified.
