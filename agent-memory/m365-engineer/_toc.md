# M365 Engineer — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| M365 Security | [platform-overview.md](platform-overview.md) | When working with Defender for Office 365, MFA, Conditional Access, Secure Score, or threat management |
| Tenant Architecture | [tenant-architecture.md](tenant-architecture.md) | When designing single/multi-tenant, Multi-Geo, or sovereign cloud configurations |
| Exchange Online | [exchange-online.md](exchange-online.md) | When working with Exchange — mail flow, connectors, transport rules, hybrid |
| SharePoint Online | [sharepoint-online.md](sharepoint-online.md) | When working with SharePoint — site types, hubs, content types, search, migration |
| Power Platform Governance | [power-platform-governance.md](power-platform-governance.md) | When governing Power Platform — environments, DLP, CoE Starter Kit |
| Intune | [intune.md](intune.md) | When managing devices — compliance policies, configuration, app deployment |
| Purview | [purview.md](purview.md) | When implementing data governance — sensitivity labels, retention, DLP, insider risk |
| Licensing | [licensing.md](licensing.md) | When optimizing M365 licensing — E3 vs E5, add-ons, audit patterns |
| Migration | [migration.md](migration.md) | When migrating to M365 from on-prem or other platforms |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing tenant for common misconfigurations — no governance, over-permissive sharing, ungoverned Power Platform |
| Gotchas | [gotchas.md](gotchas.md) | When troubleshooting unexpected behavior — license dependencies, propagation delays, policy merge logic |
| Best Practices | [best-practices.md](best-practices.md) | When designing or auditing M365 posture — security baselines, data governance, device management, monitoring |

## Identified Gaps

| Gap Topic | Priority | Notes |
|-----------|----------|-------|
| Copilot for Microsoft 365 Readiness | High | Licensing prerequisites (E3/E5 + Copilot add-on), data governance prerequisites (sensitivity labels, oversharing cleanup), adoption playbook, Copilot extensibility (plugins, Graph connectors) |
| Tenant-to-Tenant Migration | Medium | Cross-tenant mailbox migration, SharePoint cross-tenant migration, Entra ID B2B vs full migration, license transfer considerations, domain move sequencing |
