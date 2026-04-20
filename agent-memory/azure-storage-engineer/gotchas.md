# Azure Storage Gotchas

Surprising behaviors, hidden constraints, and non-obvious limitations across Azure Storage services.

## Blob Storage Gotchas

**Archive tier rehydration is not instant**:
- Standard rehydration takes up to **15 hours** — no SLA on faster completion
- High-priority rehydration targets under 1 hour for objects < 10 GB but costs significantly more
- Rehydrating to Hot costs the same as rehydrating to Cool — choose Cool if you only need temporary access
- You cannot read a blob while it's rehydrating; the tier shows "rehydrate-pending"

**Lifecycle management operates at the blob level, not container level**:
- Rules match blob prefixes and index tags — you cannot target an entire container directly
- Filters apply to block blobs only; page blobs and append blobs are excluded
- Rule evaluation runs once per day — timing is not guaranteed within a 24-hour window
- Multiple matching rules: the cheapest action wins (e.g., Archive beats Cool)

**Blob versioning creates a version on every write**:
- Even metadata-only updates create a new version
- Storage costs grow silently — easy to 10x capacity without noticing
- Must pair with lifecycle management version cleanup rules from day one
- Point-in-time restore requires versioning, soft delete, AND change feed all enabled

**CORS configuration is service-level, not per-container**:
- CORS rules apply to all containers within a service (Blob, Table, Queue, Files)
- No way to restrict CORS to a single container — design accordingly
- Maximum 5 CORS rules per service

## ADLS Gen2 Gotchas

**Hierarchical namespace changes storage account behavior permanently**:
- Cannot be disabled once enabled
- Some Blob API operations behave differently (e.g., List Blobs returns directories)
- Page blobs are not supported in HNS-enabled accounts
- Unmanaged VM disks cannot be stored in HNS accounts
- WASB driver is not supported — use ABFS driver in Hadoop environments

**ACL vs RBAC interaction is complex**:
- RBAC roles at the storage account level bypass ACLs entirely
- ACLs only apply when using Data Lake APIs, not legacy Blob APIs
- Recursive ACL application is supported but can be slow on large directory trees
- Anonymous read access overrides ACLs completely if enabled on a container

## Security & Networking Gotchas

**Each storage service needs its own private endpoint**:
- Blob, File, Table, Queue, DFS, and Web each require a separate private endpoint
- Each maps to a different privatelink DNS zone (e.g., `privatelink.blob.core.windows.net`)
- BlobFuse with HNS accounts needs both blob AND dfs private endpoints
- Forgetting one service means that service still routes over public internet

**"Allow Azure services" firewall exception is broader than expected**:
- Grants access to ANY Azure service in ANY subscription, not just yours
- Includes services like Azure DevOps, Logic Apps, and Data Factory from other tenants
- Use resource instance rules or managed identity + private endpoints instead
- This setting does not cover services running inside VMs or AKS pods

**Soft delete is per-blob, container soft delete is separate**:
- Blob soft delete protects individual blob deletions
- Container soft delete protects entire container deletions
- Both must be enabled separately — enabling one does not enable the other
- Deleted data still consumes capacity and costs money during retention period

## Azure Files Gotchas

**SMB requires port 445 — often blocked**:
- Many ISPs and corporate firewalls block outbound port 445
- No workaround except VPN, ExpressRoute, or Azure File Sync
- Test connectivity before committing to Azure Files for remote users
- Azure portal diagnostics can verify port 445 reachability

**NFS Azure Files has strict requirements**:
- Requires Premium file shares only (SSD-backed)
- Supported only on Linux clients — no Windows, no macOS
- Must use private endpoints or service endpoints (no public access)
- No identity-based authentication — relies on network-level security only

## Immutable Storage Gotchas

**Legal hold vs time-based retention differ in unlock behavior**:
- Legal hold: can be added/removed at any time by authorized users, no time expiration
- Time-based retention: once locked, interval cannot be shortened — only extended
- Unlocked time-based policies can be deleted; locked ones cannot
- Immutability applies to the container, affects all blobs within it

## Naming & Limits Gotchas

**Storage account naming is globally unique**:
- 3-24 characters, lowercase letters and numbers only (no hyphens, no uppercase)
- Name must be unique across ALL of Azure, not just your subscription
- Name becomes part of the endpoint URL (e.g., `myaccount.blob.core.windows.net`)
- Cannot be renamed after creation — must recreate and migrate data

**Storage account limits catch people off-guard**:
- Default limit: 250 storage accounts per subscription per region
- Max ingress: 25 Gbps for general-purpose v2 (can request increase)
- Max IOPS: 20,000 per standard storage account
- Single blob max throughput: ~60 MiB/s (block blob, single writer)

## Related References

- [data-protection.md](data-protection.md) — Soft delete, versioning, and immutability details
- [security.md](security.md) — Private endpoints, firewalls, and authentication
- [anti-patterns.md](anti-patterns.md) — Common storage design mistakes
- [azure-files-netapp.md](azure-files-netapp.md) — File share specifics
