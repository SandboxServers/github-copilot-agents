## Tenant Architecture

### Single Tenant vs Multi-Tenant

| Scenario | Recommendation | Rationale |
|----------|---------------|-----------|
| **Single organization, one country** | Single tenant | Simplest, lowest admin overhead |
| **Single org, global presence** | Single tenant + Multi-Geo | One tenant, data residency per geo |
| **M&A — temporary separation** | Multi-tenant + cross-tenant access | Keep separate during integration, B2B for collab |
| **Regulated subsidiaries** | Multi-tenant | Regulatory requirements mandate data isolation |
| **Sovereign cloud requirements** | Separate tenant in sovereign cloud | GCC/GCC High/DoD (US), 21Vianet (China), etc. |
| **Divestiture planned** | Multi-tenant from day one | Easier to separate later |

### Multi-Geo Considerations
- **Data residency**: satellite geo locations for Exchange, SharePoint, OneDrive, Teams
- **Licensing**: Multi-Geo requires add-on license (min 5% of total seats or 250 seats)
- **Supported geos**: varies by service — check Microsoft documentation for current list
- **Search**: unified across geos, results are security-trimmed
- **eDiscovery**: works across geos but may need separate searches

### Sovereign Clouds
| Cloud | Use Case | Key Differences |
|-------|----------|----------------|
| **Commercial** | Standard enterprise | Full feature set |
| **GCC** | US government (moderate) | US data centers, FedRAMP High |
| **GCC High** | US government (high) | DoD-level controls, ITAR/EAR data |
| **DoD** | US Department of Defense | Most restrictive, DoD-only |
| **21Vianet** | China operations | Operated by Chinese partner, separate identity |
