## Landing Zone Architecture

### Design Areas (8)

1. **Azure billing and Microsoft Entra tenant** — tenant structure, EA/MCA enrollment, billing scopes
2. **Identity and access management** — Entra ID, RBAC, PIM, conditional access, managed identity
3. **Management group and subscription organization** — hierarchy design, subscription vending
4. **Network topology and connectivity** — hub-spoke vs VWAN, DNS, ExpressRoute/VPN, private endpoints
5. **Security** — Microsoft Defender, Sentinel, key vault, DDoS protection
6. **Management** — monitoring, patching, backup, disaster recovery
7. **Governance** — Azure Policy, tags, cost management, compliance
8. **Platform automation and DevOps** — IaC, CI/CD, deployment pipelines

### Management Group Hierarchy (Reference)

```
Tenant Root Group
└─ Organization Root
   ├─ Platform
   │  ├─ Management (Log Analytics, Automation, Sentinel)
   │  ├─ Connectivity (Hub networking, DNS, Firewall)
   │  └─ Identity (Domain controllers, Entra Connect)
   ├─ Landing Zones
   │  ├─ Corp (internal workloads, private connectivity)
   │  └─ Online (internet-facing workloads)
   ├─ Sandbox (dev/test, no prod connectivity)
   └─ Decommissioned (retired subscriptions, restrictive policies)
```

### Platform vs Application Landing Zones

| Type | Who Manages | Purpose | Examples |
|------|------------|---------|----------|
| Platform LZ | Central IT / Platform team | Shared services foundation | Connectivity, identity, management |
| Application LZ (Central) | Central IT operates for app teams | Standard workload hosting | Internal LOB apps |
| Application LZ (Delegated) | App team self-manages within guardrails | Team autonomy with governance | DevOps teams, product teams |
| Application LZ (Shared) | Split: platform team + app team | Technology platforms | AKS, AVS, SAP |
