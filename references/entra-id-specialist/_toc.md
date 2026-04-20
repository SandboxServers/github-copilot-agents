# Entra ID Specialist — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Authentication Flows | [authentication-flows.md](authentication-flows.md) | When designing OAuth 2.0/OIDC/SAML flows or choosing auth patterns |
| App Registrations | [app-registrations.md](app-registrations.md) | When working with app registrations, enterprise apps, or permissions |
| Conditional Access | [conditional-access.md](conditional-access.md) | When designing CA policies, layering, break-glass, or troubleshooting |
| Workload Identity | [workload-identity.md](workload-identity.md) | When implementing secretless auth for GitHub Actions, AKS, or Terraform |
| PIM | [pim.md](pim.md) | When implementing just-in-time access, access reviews, or role governance |
| Hybrid Identity | [hybrid-identity.md](hybrid-identity.md) | When designing Entra Connect, Cloud Sync, PHS/PTA, or AD FS migration |
| B2B & B2C | [b2b-b2c.md](b2b-b2c.md) | When designing guest access, customer identity, or external collaboration |
| Service Principal Hygiene | [service-principal-hygiene.md](service-principal-hygiene.md) | When auditing service principals — credentials, expiry, permissions, cleanup |
| Tenant Security | [tenant-security.md](tenant-security.md) | When hardening tenant security — admin accounts, MFA, ID Protection |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing Entra ID misconfigurations — over-permissioned apps, missing CA, secret sprawl |
| Gotchas | [gotchas.md](gotchas.md) | When debugging subtle Entra behaviors — app reg vs SP, CA evaluation, token lifetimes |
| Best Practices | [best-practices.md](best-practices.md) | When planning Entra ID security posture — CA baseline, PIM, workload identity, monitoring |

## Gap Analysis

Existing references cover core Entra ID areas well. Potential future additions:

| Gap Area | Notes |
|----------|-------|
| Entra ID Governance | Entitlement management, access packages, lifecycle workflows — partially touched in PIM but deserves dedicated file |
| Entra Verified ID | Decentralized identity / verifiable credentials — emerging area, not yet covered |
| Entra Permissions Management (CIEM) | Multi-cloud permissions analytics (Azure, AWS, GCP) — separate product, not covered |
| Global Secure Access | Entra Private Access + Internet Access (ZTNA replacement for VPN) — not covered |
| B2C → External ID Migration | Migration path from legacy B2C to Entra External ID — briefly noted in gotchas, deserves dedicated guidance |
| Hybrid Identity Deep Dive | Password writeback, seamless SSO troubleshooting, staged rollout — hybrid-identity.md could be expanded |
