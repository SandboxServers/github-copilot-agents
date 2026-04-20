# Well-Architected: Reliability & Security

## Reliability
### Availability Zones
Most Azure services support zone redundancy. Use it for production.
- App Service: Premium v3 with zone redundancy (min 3 instances)
- AKS: Zone-redundant node pools (nodes spread across zones)
- SQL Database: Zone-redundant option on Premium/Business Critical
- Storage: ZRS or GZRS replication
- Key Vault: Automatically zone-redundant in supported regions

### Multi-Region
When single-region isn't enough (RPO/RTO requirements):
- Azure Front Door for global routing and failover
- Cosmos DB with multi-region writes
- Azure SQL with failover groups (auto-failover, readable secondaries)
- Traffic Manager for DNS-based routing (slower failover)

### Resiliency Patterns
- **Retry with backoff**: Use Polly (.NET), tenacity (Python). Exponential backoff + jitter.
- **Circuit breaker**: Stop calling a failing dependency. Half-open state to test recovery.
- **Bulkhead**: Isolate failures. Separate connection pools per dependency.
- **Queue-based load leveling**: Service Bus between frontend and backend. Absorb traffic spikes.
- **Health endpoints**: /health endpoint on every service. Used by load balancers, Container Apps probes, AKS probes.

### SLA Composition
Composite SLA = SLA₁ × SLA₂ × SLA₃
Example: App Service (99.95%) × SQL DB (99.99%) × Key Vault (99.99%) = 99.93%
Add redundancy to improve: Two regions with Front Door ≈ 99.9999%

## Security
### Defense in Depth (Layers)
1. Identity: Entra ID, MFA, Conditional Access
2. Network: NSGs, private endpoints, WAF, DDoS Protection
3. Compute: Patched images, Defender for Containers/Servers
4. Application: Input validation, HTTPS only, CORS
5. Data: Encryption at rest (SSE, TDE), encryption in transit (TLS 1.2+), RBAC

### Private Endpoints (Production Standard)
Every data service should use private endpoints in production:
Storage, SQL Database, Cosmos DB, Key Vault, Redis Cache, Service Bus, Event Hubs, Azure OpenAI, Container Registry

### Secrets Management
- NEVER in code, config files, or environment variables
- Key Vault for secrets, certificates, keys
- App Service Key Vault references for app settings
- Managed identity to access Key Vault (no connection string needed)
