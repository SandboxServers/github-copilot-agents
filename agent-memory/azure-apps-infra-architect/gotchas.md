# Azure Architecture Gotchas

Platform-specific surprises that catch teams during implementation or production.

> **Source:** Microsoft Learn documentation, Azure service limits, real-world deployment issues.

## Compute Gotchas

### App Service Plan: Shared Compute = Noisy Neighbor
Multiple apps on one App Service Plan share the same compute. One runaway app starves the others. Scaling applies to the plan, not individual apps. Isolate high-traffic or CPU-intensive apps on dedicated plans.

### Container Apps vs AKS: Networking Limits
Container Apps has a simpler networking model but with constraints. Custom VNet integration requires a dedicated subnet with a /23 or larger CIDR. No support for custom DNS servers within the environment. No Windows containers. If you need full Kubernetes networking (service meshes, custom CNI, network policies), use AKS.

### Azure Functions Consumption Cold Start
Consumption plan instances are deallocated after ~20 minutes of inactivity. Cold starts range from 1-10+ seconds depending on runtime and dependencies. Not acceptable for latency-sensitive APIs. Mitigations: Flex Consumption plan with always-ready instances, Premium plan with pre-warmed workers, or keep-alive pings (not recommended).

### App Service Deployment Slots Share Plan Resources
Slots are NOT additional compute. A staging slot runs on the same App Service Plan instances as production. Load testing a staging slot impacts production performance. Use a separate plan for heavy pre-production testing.

## Networking Gotchas

### Private Endpoints: DNS Is the #1 Failure Mode
Enabling private endpoints without correct DNS configuration is the most common failure. You need: (1) a private DNS zone linked to the VNet, (2) the A record pointing to the private IP, (3) conditional forwarding if using custom DNS servers. Test with `nslookup` — if it resolves to the public IP, DNS is wrong.

### VNet Integration for App Service: Subnet Requirements
Regional VNet integration requires a dedicated, empty subnet. The subnet size determines the max instances — each instance consumes one IP. A /26 gives ~60 IPs (usable for scaling). Cannot share the subnet with other resources. Subnet delegation to `Microsoft.Web/serverFarms` is required.

### Azure Front Door vs Application Gateway: Different Routing
Front Door operates at the global edge (Anycast) with URL path-based routing evaluated globally. Application Gateway is regional with more granular routing rules (header-based, query string). They serve different purposes — Front Door for global load balancing and WAF, Application Gateway for regional L7 routing. Don't pick one thinking it replaces the other.

### Private Endpoints Block Public Access (Sometimes)
Enabling a private endpoint on a storage account or database doesn't automatically disable public access in all cases. You must explicitly set `publicNetworkAccess: Disabled` or configure firewall rules. Default behavior varies by service.

## Messaging & Integration Gotchas

### Service Bus Premium vs Standard: Partitioning Changes
Standard tier uses partitioned entities by default (16 partitions). Premium tier does NOT support partitioning — entities run on dedicated resources. Migrating from Standard to Premium may change message ordering behavior if you relied on partition-level ordering.

### Event Grid vs Service Bus: Different Delivery Guarantees
Event Grid provides at-least-once delivery with best-effort ordering. Service Bus provides at-least-once with FIFO ordering (sessions). Pick the right tool: Event Grid for event notifications, Service Bus for reliable command/workflow processing.

## Platform Gotchas

### Availability Zones: Not Universal
Not all Azure services support Availability Zones in all regions. Zone-redundant deployments require explicit opt-in — they are not automatic. Check the [Azure services with Availability Zone support](https://learn.microsoft.com/azure/reliability/availability-zones-service-support) matrix before designing HA architectures.

### Resource Group Deletion: No Partial Undo
Deleting a resource group removes ALL resources within it. There is no recycle bin, no partial deletion, no undo. Structure resource groups by lifecycle — resources that live and die together. Never put production databases and ephemeral resources in the same group.

### ARM Deployment History: 800 Limit
Each resource group retains a history of 800 deployments. When you hit the limit, new deployments fail. Azure auto-deletes old deployments (opt-in), or clean them up in CI/CD pipelines. Monitor deployment count in long-lived resource groups.

### Subscription Limits Are Real
Azure subscriptions have limits: 980 resource groups, region-specific vCPU quotas, storage accounts per region (250 default). These are soft limits — request increases proactively, not during an outage.

## Data & Storage Gotchas

### Cosmos DB RU Pricing Surprises
Cosmos DB charges per Request Unit (RU). Cross-partition queries, large document reads, and indexing all consume RUs. A poorly designed partition key can cause hot partitions and throttling. Use the Cosmos DB capacity calculator and test with realistic workloads.

### Azure SQL DTU vs vCore: Can't Switch Easily
Choosing between DTU and vCore models at creation time has implications. While you can migrate between them, it requires downtime or a new database. vCore is recommended for new workloads — it gives transparent resource allocation and Azure Hybrid Benefit eligibility.

## Quick Reference

| Gotcha | Impact | Mitigation |
|--------|--------|------------|
| App Service shared plan | Noisy neighbor | Separate plans for critical apps |
| Functions cold start | Latency spikes | Flex Consumption or Premium plan |
| Private endpoint DNS | Connection failures | Private DNS zones + validation |
| VNet integration subnet | Scaling limits | Size subnet for max instances |
| Resource group deletion | Data loss | Lifecycle-aligned resource groups |
| ARM 800 deployment limit | Deployment failures | Enable auto-delete or CI/CD cleanup |
| Cosmos DB RU model | Cost overruns | Capacity calculator + monitoring |
| AZ support varies by region | HA gaps | Check service/region matrix |
