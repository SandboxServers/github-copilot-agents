# Azure Architecture Anti-Patterns

Common mistakes that lead to outages, overspend, and operational pain.

> **Source:** Azure Well-Architected Framework, Microsoft Learn best practices, real-world failure patterns.

## Reliability Anti-Patterns

### Single Region Without DR Plan
Deploying production workloads in one region without a failover strategy. Azure regions do experience outages — even paired regions are not immune. Always define RTO/RPO and have a tested recovery plan.

### No Autoscaling — Fixed Compute for Variable Workloads
Provisioning fixed compute (e.g., static App Service instances) for workloads with variable traffic. Results in either wasted spend during low periods or outages during peaks. Use autoscale rules, KEDA-based scaling, or consumption plans.

### Missing Health Endpoints and Readiness Probes
Deploying services without `/health` and `/ready` endpoints. Load balancers, orchestrators, and monitoring tools need these to route traffic correctly and detect failures. Without them, unhealthy instances receive traffic.

### No Retry Logic or Circuit Breakers
Assuming all inter-service calls succeed on the first attempt. Transient failures are normal in distributed systems. Use Polly (.NET), resilience4j (Java), or built-in SDK retry policies. Implement circuit breakers to prevent cascading failures.

### Tight Coupling Between Services
Services that directly depend on each other's availability — synchronous call chains where one failure cascades through the entire stack. Use asynchronous messaging (Service Bus, Event Grid), event-driven patterns, and bulkhead isolation.

## Security Anti-Patterns

### Publicly Exposed Management Endpoints
Exposing admin panels, Kudu/SCM sites, or management APIs to the public internet. Always restrict management endpoints to VPN, private networks, or IP-allowlisted access.

### No Network Segmentation (Flat Network Topology)
Deploying all resources in a single subnet or VNet without segmentation. Compromising one resource gives lateral access to everything. Use NSGs, subnets per tier, and private endpoints for data services.

### Secrets in App Settings or Code
Hardcoding connection strings, API keys, or passwords in app configuration or source code. Use Key Vault references, managed identity, and workload identity federation. Zero secrets in config.

## Cost Anti-Patterns

### Cost Blindness — Designing Without Considering Operational Costs
Architects who pick services without understanding the pricing model. Cosmos DB RU overprovisioning, Premium SKUs in dev/test, idle AKS clusters running 24/7. Review pricing calculators during design, set budgets, and use cost alerts.

### Over-Engineering — Complex Services When Simple Ones Suffice
Choosing AKS for a simple CRUD API. Deploying API Management for two internal endpoints. Using Cosmos DB when Azure SQL would work. Match service complexity to actual requirements. Start simple, evolve when proven needed.

### Premium Everything in Non-Production
Running Premium v3 App Service Plans, Premium Service Bus, and Standard SSD disks in dev/test environments. Use Basic/Standard SKUs, auto-shutdown schedules, and dev/test pricing where available.

## Operational Anti-Patterns

### Monolithic Deployments — All-or-Nothing Releases
Deploying the entire application as a single unit. One bad change takes down everything. Use deployment slots, canary releases, feature flags, and independent service deployments.

### Not Using Managed Services
Running PostgreSQL, Redis, or RabbitMQ on VMs instead of using Azure Database for PostgreSQL, Azure Cache for Redis, or Service Bus. You inherit patching, backup, HA, and scaling responsibilities. PaaS eliminates undifferentiated heavy lifting.

### Manual Deployments via Azure Portal
Click-ops for production infrastructure. No audit trail, no reproducibility, no rollback. Use Bicep, Terraform, or ARM templates for all infrastructure. CI/CD for all application deployments.

### Ignoring WAF Pillars During Design
Treating the Well-Architected Framework as an afterthought. WAF pillars (Reliability, Security, Cost Optimization, Operational Excellence, Performance Efficiency) should be evaluation criteria during design, not a post-deployment audit.

## Data Anti-Patterns

### Shared Databases Across Services
Multiple services reading/writing the same database. Schema changes become cross-team coordination nightmares. Deployments are coupled. Use database-per-service with well-defined APIs or events for data sharing.

### No Data Residency Planning
Deploying globally without considering data sovereignty requirements. GDPR, HIPAA, and industry regulations mandate where data can reside. Plan data residency from day one, not as a retrofit.

## Summary Checklist

| Category | Anti-Pattern | Quick Fix |
|----------|-------------|-----------|
| Reliability | Single region | Multi-region + Front Door |
| Reliability | No autoscaling | Autoscale rules or consumption plans |
| Reliability | No health probes | Add /health and /ready |
| Reliability | No retry/circuit breaker | Polly, SDK retries, circuit breakers |
| Security | Public management endpoints | IP restrictions, private access |
| Security | Flat network | Subnets, NSGs, private endpoints |
| Security | Secrets in config | Key Vault + managed identity |
| Cost | Cost blindness | Pricing review during design |
| Cost | Over-engineering | Right-size service selection |
| Operations | Monolithic deploys | Slots, canary, feature flags |
| Operations | Manual deployments | IaC + CI/CD |
| Operations | Skip WAF pillars | WAF review during design |
| Data | Shared databases | Database per service |
