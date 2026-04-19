# Security

## Defense in Depth for Data Platforms

```
Layer 1: Network
├─ Private endpoints for all data services
├─ VNet integration for Spark clusters
├─ No public endpoints in production
└─ NSGs restricting data plane traffic

Layer 2: Identity
├─ Managed identity for all service-to-service auth
├─ Entra ID for all user authentication
├─ No shared accounts, no service account passwords
└─ PIM for elevated access to data services

Layer 3: Authorization
├─ Unity Catalog / Purview for fine-grained access
├─ RBAC roles scoped to minimum necessary
├─ Row-level security for multi-tenant data
├─ Column-level masking for sensitive fields
└─ Dynamic data masking in SQL endpoints

Layer 4: Data Protection
├─ Encryption at rest (platform-managed or CMK)
├─ Encryption in transit (TLS 1.2+)
├─ Sensitivity labels propagated from Purview
├─ Immutable storage for audit/compliance data
└─ Customer-managed keys for regulated workloads

Layer 5: Monitoring
├─ Diagnostic settings on all resources
├─ Audit logs for data access
├─ Anomaly detection for unusual access patterns
└─ Cost alerts for unexpected usage spikes
```
