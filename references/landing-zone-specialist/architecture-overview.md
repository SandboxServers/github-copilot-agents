## Landing Zone Architecture at a Glance

```
Tenant Root Group
└─ Organization Root (top-level MG — policies inherited by ALL)
   ├─ Platform (shared services — managed by platform team)
   │  ├─ Management        → Log Analytics, Automation, Sentinel, Update Management
   │  ├─ Connectivity      → Hub networking, DNS zones, Firewall, ExpressRoute/VPN
   │  └─ Identity          → Domain controllers, Entra Connect servers
   ├─ Landing Zones (workloads — delegated to app teams with guardrails)
   │  ├─ Corp              → Internal workloads, private connectivity, no public endpoints
   │  └─ Online            → Internet-facing workloads, public endpoints allowed
   ├─ Sandbox              → Dev experimentation, no prod connectivity, relaxed policies
   └─ Decommissioned       → Retired subscriptions, deny-all policies
```

### 8 Design Areas

| # | Design Area | Key Decisions | Hard to Change Later? |
|---|------------|---------------|----------------------|
| 1 | **Billing & Tenant** | EA/MCA enrollment, tenant structure, billing scopes | YES — tenant restructuring is painful |
| 2 | **Identity & Access** | Entra ID, RBAC model, PIM, conditional access, managed identity | YES — identity architecture changes ripple everywhere |
| 3 | **Management Groups & Subscriptions** | MG hierarchy, subscription organization, vending process | MODERATE — MG moves are possible but disruptive |
| 4 | **Network Topology** | Hub-spoke vs VWAN, DNS strategy, ExpressRoute/VPN, private endpoints | YES — network topology changes are the hardest |
| 5 | **Security** | Defender for Cloud, Sentinel, Key Vault, DDoS protection | MODERATE — can be added incrementally |
| 6 | **Management** | Monitoring, patching, backup, disaster recovery | MODERATE — can iterate |
| 7 | **Governance** | Azure Policy, tags, cost management, compliance | MODERATE — policies can be phased in |
| 8 | **Platform Automation** | IaC tooling, CI/CD, deployment pipelines | MODERATE — tooling can evolve |
