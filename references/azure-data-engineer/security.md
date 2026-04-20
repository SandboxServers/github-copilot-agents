# Data Platform Security

Defense-in-depth security model for Azure data platforms.

## Network Security

- Private endpoints for all data services: ADLS Gen2, Azure SQL, Synapse, Cosmos DB
- Managed VNet for Fabric workspaces and Databricks clusters
- No public access for production data platforms — private connectivity only
- VNet injection for Databricks workers (data plane in customer VNet)
- NSGs restricting data plane traffic between subnets
- Azure Private Link for cross-region and cross-subscription data access
- DNS private zones for name resolution of private endpoints
- Service endpoints as minimum baseline when private endpoints aren't feasible

## Identity and Authentication

- Managed identity for all compute-to-data-store authentication
- Service principals for CI/CD pipelines only — not for interactive use
- Entra ID passthrough for user-level security in Databricks (identity propagation)
- No shared accounts and no service account passwords stored in config
- Privileged Identity Management (PIM) for elevated access to data services
- Conditional Access policies for data platform portal and API access
- Multi-factor authentication required for all human identities

## Encryption

- At rest: server-side encryption with Microsoft-managed keys (default)
- Customer-managed keys (CMK) in Key Vault for regulated workloads
- In transit: TLS 1.2+ enforced for all data platform connections
- Transparent Data Encryption (TDE) for SQL databases and Synapse pools
- Double encryption available for ADLS Gen2 (infrastructure + service layer)
- Encryption scope in ADLS Gen2 for per-container key management

## Access Control

- RBAC at workspace and resource level for administrative operations
- Table ACLs in Unity Catalog for fine-grained data access (GRANT/REVOKE)
- Row-level security (RLS) for multi-tenant data filtering by user context
- Column masking for PII: dynamic masking shows partial data to non-privileged users
- Column-level security: restrict access to sensitive columns entirely
- Fabric workspace roles: Admin, Member, Contributor, Viewer with scoped permissions
- OneLake data access roles for folder-level permissions in lakehouses

## Key Management

- Azure Key Vault for all secrets, keys, and certificates
- Databricks secret scopes backed by Key Vault (unified secret management)
- Fabric uses workspace identity for managed credential propagation
- Key rotation policies: automate key rotation on schedule
- Access policies or RBAC on Key Vault — never give broad access
- Separate Key Vaults for dev/staging/production environments

## Auditing and Monitoring

- Diagnostic settings enabled on all data services → Log Analytics workspace
- Purview audit logs for data discovery and classification events
- Databricks audit logs: workspace events, SQL access, cluster operations
- Activity logs for pipeline runs, copy operations, data transformations
- Microsoft Sentinel integration for security event correlation and alerting
- Anomaly detection for unusual data access patterns (volume, time, source IP)
- Cost alerts for unexpected usage spikes that may indicate compromised credentials
- Retention: 90-day minimum for audit logs, longer for compliance requirements
