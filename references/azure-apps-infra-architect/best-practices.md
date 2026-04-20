# Azure Architecture Best Practices

Foundational practices for production Azure architectures.

> **Source:** Azure Well-Architected Framework, Cloud Adoption Framework, Microsoft Learn reference architectures.

## Well-Architected Framework Alignment

Every architecture decision should be evaluated against the five WAF pillars:

| Pillar | Key Question | Default Action |
|--------|-------------|----------------|
| Reliability | "What happens when this fails?" | Multi-region, health probes, retry policies |
| Security | "Who can access this and how?" | Zero Trust, managed identity, private endpoints |
| Cost Optimization | "Are we paying for what we use?" | Right-size SKUs, auto-shutdown, reservations |
| Operational Excellence | "Can we deploy and operate this safely?" | IaC, CI/CD, monitoring, runbooks |
| Performance Efficiency | "Does this scale to meet demand?" | Autoscale, caching, CDN, async patterns |

## Networking

### Hub-Spoke as Default Starting Point
Use hub-spoke network topology for all non-trivial architectures. Hub VNet hosts shared services (firewall, DNS, VPN gateway). Spoke VNets host workloads with peering to the hub. Provides centralized security, network segmentation, and simplified management.

### Private Endpoints for All Data Services
Storage accounts, databases, Key Vaults, and Service Bus should use private endpoints. No data service should be reachable from the public internet in production. Combine with private DNS zones for name resolution.

### Network Segmentation by Tier
Separate subnets for web tier, application tier, and data tier. NSGs restrict traffic between tiers. Only the web tier is exposed via a load balancer or Application Gateway.

## Identity & Security

### Managed Identity Everywhere
Eliminate connection strings, API keys, and passwords from application configuration. Use system-assigned managed identity for single-resource scenarios. Use user-assigned managed identity when multiple resources need the same identity. Workload identity federation for CI/CD (GitHub Actions, Azure Pipelines).

### Zero Trust Network Access
Assume breach. Verify explicitly. Use least-privilege access. Combine with Conditional Access policies, Just-In-Time VM access, and Privileged Identity Management for administrative access.

## Reliability

### Zone-Redundant Deployments for Production
All production workloads should be zone-redundant where the service supports it. App Service Premium v3, AKS with zone-aware node pools, zone-redundant SQL Database, and ZRS storage accounts. This protects against datacenter-level failures within a region.

### Health Endpoints from Day One
Every service must expose `/health` (liveness) and `/ready` (readiness) endpoints. Health checks should verify downstream dependencies. Azure Load Balancer, Application Gateway, and Front Door all use health probes for routing decisions.

### Retry and Circuit Breaker Patterns
All inter-service communication must include retry logic with exponential backoff and jitter. Circuit breakers prevent cascading failures when a downstream service is degraded. Use Polly (.NET), resilience4j (Java), or equivalent libraries.

## Deployment & Operations

### Infrastructure as Code: Bicep or Terraform
All infrastructure must be defined in code. Bicep for Azure-only (first-party, ARM-native). Terraform for multi-cloud or teams with existing HCL expertise. Never create production resources through the portal. Use `what-if` / `plan` before every deployment.

### Blue-Green and Canary Deployments
Use deployment slots (App Service), revision-based traffic splitting (Container Apps), or rolling updates (AKS) for zero-downtime deployments. Never deploy directly to production. Always have a rollback path.

### Monitoring from Day One
Deploy Application Insights and Log Analytics workspace with the first resource. Configure alerts for error rates, response times, and resource saturation. Use availability tests for external endpoint monitoring. Dashboards for operational visibility.

## Cost Management

### Tag Everything
Every resource must have tags: `environment`, `team`, `cost-center`, `application`, `created-by`. Tags enable cost allocation, automated governance (Azure Policy), and lifecycle management. Enforce tagging via Azure Policy deny rules.

### Right-Size from the Start
Start with the smallest viable SKU and scale up when metrics justify it. Use Azure Advisor recommendations for right-sizing. Dev/test environments should use Basic/Standard SKUs with auto-shutdown schedules. Reservations and savings plans for predictable production workloads.

### Budget Alerts and Cost Analysis
Set Azure budget alerts at 50%, 75%, 90%, and 100% of expected spend. Use Cost Analysis to identify top cost drivers. Review costs weekly during development, monthly in production.

## Summary Checklist

| Practice | Priority | Implementation |
|----------|----------|---------------|
| Hub-spoke networking | P0 | VNet design, peering, centralized firewall |
| Managed identity | P0 | System/user-assigned MI, Key Vault integration |
| Zone-redundant production | P0 | Premium SKUs, ZRS, zone-aware AKS |
| IaC (Bicep/Terraform) | P0 | All resources in code, CI/CD pipelines |
| Health endpoints | P0 | /health and /ready on every service |
| Monitoring + alerts | P0 | App Insights, Log Analytics, alert rules |
| Tag everything | P1 | Policy-enforced tagging standards |
| Blue-green/canary deploys | P1 | Slots, traffic splitting, rolling updates |
| Retry + circuit breakers | P1 | Polly/.NET, SDK retry policies |
| Cost budgets + alerts | P1 | Budget alerts, weekly cost review |
