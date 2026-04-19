## Common Anti-Patterns the Specialty Lead Catches

### Local Optimization at Global Expense
- Database specialist picks Cosmos DB for global distribution, but compute engineer deploys single-region App Service
- Storage engineer puts data in archive tier for cost savings, but data engineer needs sub-second access for pipelines
- AI specialist requests GPU compute in region with no ExpressRoute connectivity

### Missing Cross-Cutting Concerns
- Private endpoints configured for database but not for storage
- Managed identity for App Service but not for Function Apps in the same solution
- Application Insights on the web tier but no diagnostic settings on the database tier

### Sequencing Failures
- Deployed compute before network topology was finalized — now VNet integration requires redeployment
- Built data pipelines before database partitioning strategy was decided — now reshuffling data
- Added monitoring after production issues — now missing historical baseline data

### Technology Selection Mismatches
- Using Service Bus for simple async when Storage Queue would suffice (and cost less)
- Using AKS when Container Apps would handle the workload with less operational overhead
- Using Cosmos DB for a workload that's purely relational with no global distribution need
