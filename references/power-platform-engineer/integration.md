## Integration

### Custom Connectors
- Wrap any REST API as a Power Platform connector
- Define operations, authentication (OAuth 2.0, API Key, Basic), request/response schemas
- Supports OpenAPI (Swagger) definition import
- Share within environment or package in solutions for ALM
- **Certified connectors**: publish to the public connector gallery (Microsoft certification process)

### Azure Functions Integration
- Call Azure Functions via HTTP connector or custom connector
- Good for complex logic that doesn't belong in a flow (calculations, external API orchestration)
- Use **managed identity** for auth between Power Platform and Azure resources
- Keep functions stateless and idempotent for flow retries

### On-Premises Data Gateway
- Bridge between Power Platform cloud services and on-premises data sources
- Required for: on-prem SQL Server, file shares, Oracle, SAP, Analysis Services
- Install on a machine that can reach both the internet and the on-prem data source
- **Gateway cluster** — install on multiple machines for HA and load balancing
- **Premium connectors + gateway = double licensing requirement** (premium license + gateway infra)
