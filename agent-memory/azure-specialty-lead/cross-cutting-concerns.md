## Cross-Cutting Concerns

Cross-cutting concerns span all specialties. No single specialist owns them, but the Specialty Lead ensures they are consistent across every domain. Inconsistency in cross-cutting concerns is the most common source of production security gaps and operational confusion.

### Identity

Identity is the security foundation. Inconsistent identity is a security incident waiting to happen.

- **Managed identity everywhere.** Every Azure resource that supports managed identity must use it. No connection strings with embedded credentials. No shared keys in app settings.
- **RBAC consistency.** Same role model across all resources. If compute uses Contributor on one resource group and Owner on another, something is wrong.
- **Key Vault for secrets.** When managed identity isn't possible (third-party services, cross-tenant), secrets go in Key Vault. Key Vault references in App Service and Function App configuration — not environment variables with plain-text secrets.
- **Entra ID for human access.** No shared accounts, no local admin passwords stored in wikis. PIM for elevated access. Conditional access policies for all administrative portals.

### Networking

Networking decisions constrain every other domain. Get networking wrong and everything else is a workaround.

- **Private endpoints for all PaaS.** Storage, databases, Key Vault, App Service (when applicable), Container Registry — all behind private endpoints. Public endpoints disabled or restricted to known IPs during development only.
- **Consistent DNS strategy.** Private DNS zones linked to hub VNet (or DNS resolver). Every specialist must use the same DNS resolution path. Split-brain DNS causes mysterious intermittent failures.
- **Network segmentation.** Subnets with NSGs. Workload isolation. Database subnet separate from compute subnet. Management subnet for bastion and DevOps agents.
- **Egress control.** Azure Firewall or NVA for egress filtering in production. Not just "allow all outbound." Know what your workloads talk to externally.

### Monitoring

What you can't observe, you can't troubleshoot. Monitoring must be designed alongside the solution, not bolted on after the first production incident.

- **Diagnostic settings on every resource.** Every Azure resource must send logs and metrics to a centralized Log Analytics workspace. No exceptions. No "we'll add it later."
- **Application Insights on every compute.** Web apps, APIs, Function Apps, container workloads — all instrumented with Application Insights. Distributed tracing enabled.
- **Alerts for every critical path.** Not just CPU and memory. Availability, latency P95, error rates, queue depth, database DTU consumption, storage throttling.
- **Unified dashboards.** One place to see system health across all domains. Not seven specialist dashboards that nobody correlates.

### Security

Security is not a phase — it's a constraint on every decision.

- **Encryption at rest and in transit.** TLS 1.2+ for all connections. Customer-managed keys where compliance requires it. Platform-managed keys as default.
- **Least privilege.** Every identity — human or managed — gets the minimum permissions required. Review RBAC assignments quarterly.
- **WAF on all public endpoints.** Azure Front Door or Application Gateway with WAF policy. OWASP rule sets enabled. Custom rules for application-specific threats.
- **Vulnerability scanning.** Defender for Cloud enabled on all subscriptions. Container image scanning. Dependency scanning in CI/CD.

### Tagging

Tags are how you answer "who owns this?" and "how much does this cost?" after deployment.

- **Consistent taxonomy.** Every resource tagged with: `environment`, `owner`, `cost-center`, `application`, `created-by`. No variations (`env` vs `environment` vs `Environment`).
- **Azure Policy enforcement.** Deny deployment of resources missing required tags. Inherit tags from resource group where appropriate.
- **Tag-based cost allocation.** Cost Management reports grouped by tags. Monthly review with cost center owners.

### Naming Conventions

Names should be predictable. Anyone looking at a resource should know what it is, where it is, and what it belongs to.

- **Pattern:** `{resource-type}-{application}-{environment}-{region}-{instance}`
- **Examples:** `vnet-platform-prod-eastus2`, `sql-orders-staging-westus3`, `kv-shared-prod-eastus2-001`
- **Abbreviations:** Use Azure CAF recommended abbreviations (`rg`, `vnet`, `kv`, `sql`, `st`, `cr`, `aks`)
- **Enforcement:** Azure Policy to audit naming convention compliance. Fix violations before they proliferate.

### The Specialty Lead's Role

Each specialist applies cross-cutting concerns within their domain. The Specialty Lead ensures they apply them **consistently** across all domains. Private endpoints on databases but not storage is a gap. Managed identity on App Service but not Functions is a gap. Monitoring on compute but not on integration is a gap. The Specialty Lead finds and closes these gaps.
