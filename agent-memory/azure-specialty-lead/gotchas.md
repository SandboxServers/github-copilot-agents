## Gotchas in Azure Specialty Coordination

These are the non-obvious traps that catch even experienced teams. They don't look like problems until they are.

### Cross-Cutting Concerns Touch Everything

Identity, monitoring, and networking are not isolated domains — they are woven through every other domain. A change in any cross-cutting concern ripples everywhere.

- **Identity model change** (switching from service principals to managed identities) requires changes in compute, database, storage, integration, and monitoring configurations — all at once, not sequentially.
- **Network topology change** (adding a firewall, changing DNS resolution, enabling private endpoints) affects every service that communicates. One missed route and a service goes dark.
- **Monitoring agent update** (new agent version, different collection endpoint) touches every compute resource. Miss one and you have an observability gap.

**Reality:** Cross-cutting changes are never "just" one domain. Budget time for cross-domain validation on every cross-cutting change.

### Regional Availability Gaps

Not all Azure services are available in all regions. Not all SKUs are available in all regions. Not all features of a service are available in all regions.

- Specialist designs a solution using a preview feature available in East US. The deployment target is Brazil South. The feature doesn't exist there.
- Availability Zones exist in most regions but not all. A reliability design assuming AZs breaks in regions without them.
- Capacity constraints: even generally-available services may have capacity limits in specific regions during high-demand periods.

**Reality:** Validate regional availability for every service, SKU, and feature before design is finalized. Use `az provider show` or the Azure documentation region-by-region tables.

### Service Limits Outside Your Domain

Each specialist knows their own domain's limits intimately. They rarely know the limits of adjacent domains that their design depends on.

- Database specialist designs for 100K RPS. Network hasn't checked that the NSG rule evaluation rate or the load balancer can handle that throughput.
- Integration specialist plans 500 concurrent Logic App runs. Nobody checked whether the downstream API's App Service plan can absorb that load.
- Compute specialist scales to 200 VM instances. Storage specialist didn't know that the storage account has a 20K RPS limit per partition.

**Reality:** Every cross-domain design must include a limits review. Each specialist validates that adjacent domains can handle their throughput, concurrency, and scale assumptions.

### Dependency Chain Fragility

Changing one foundational service cascades through everything above it. The longer the chain, the more fragile the system.

- Changing the virtual network address space requires re-peering, updating route tables, reconfiguring firewalls, updating DNS records, and restarting services that cache DNS. One missed step and connectivity drops.
- Changing the identity provider (e.g., migrating from B2C to Entra External ID) touches every application's authentication flow simultaneously.
- Changing the logging sink (e.g., moving from one Log Analytics workspace to another) breaks every alert, dashboard, and workbook that queries the old workspace.

**Reality:** Map dependency chains before making foundational changes. Identify every downstream consumer. Plan rollback for each link in the chain.

### Cost Accumulation from "Small Additions"

Each specialist adds something small and justified. The database team adds read replicas. The compute team adds an extra node for headroom. The network team adds DDoS Protection Standard. The monitoring team enables all diagnostic logs. Individually, each is a rounding error. Together, they double the bill.

- Premium tiers compound: Premium storage + Premium compute + Premium database + Premium networking = 4x baseline cost.
- Redundancy compounds: Zone-redundant everything across three domains is 3x the single-zone cost per domain.
- Logging compounds: verbose diagnostic settings on every resource generates terabytes of logs nobody reads.

**Reality:** Track cost incrementally. Every design change gets a cost delta. Review cumulative cost at every architecture checkpoint.

### Azure Service Updates Break Assumptions

Azure evolves constantly. A design validated today may have different constraints tomorrow.

- API versions deprecate. A specialist's automation scripts stop working when an older API version is retired.
- Default behaviors change. A new storage account default (e.g., requiring TLS 1.3) breaks older clients that another specialist's workload depends on.
- Feature GA changes behavior from preview. A feature that worked one way in preview may have different limits, pricing, or behavior at general availability.
- Service retirements. Azure retires services (Classic VMs, ASM, ACS). Designs built on retiring services need migration plans that span multiple domains.

**Reality:** Subscribe to Azure Updates. Review service changes monthly. Maintain a dependency register that maps which services and API versions each domain depends on.
