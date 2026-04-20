## Service Principal Hygiene

### Audit Priorities

Service principals are the most under-managed identity type in most tenants.
Regular audits prevent credential leaks, over-permissioning, and outages.

### Credential Management

**Certificate credentials (preferred)**:
- Asymmetric key — private key never leaves the workload
- Cannot be accidentally logged or leaked as easily as secrets
- Longer validity acceptable (1-2 years) with rotation plan
- Upload public key to app registration, workload holds private key

**Client secrets (avoid long-lived)**:
- Symmetric key — the secret itself is the credential
- Can be leaked in logs, repos, environment variables
- Set maximum expiry: 6 months or shorter
- Never use secrets with no expiry or multi-year expiry
- If secrets are required, automate rotation

### Expiry Monitoring

- Alert at least 30 days before any credential expires
- Track all credentials across app registrations in a central inventory
- Automate expiry checks via Microsoft Graph:
  - `GET /applications` → `passwordCredentials`, `keyCredentials`
  - Filter for `endDateTime` approaching
- Integrate alerts into operational monitoring (Sentinel, ServiceNow, PagerDuty)

### Unused Service Principals

- No sign-in activity for 90+ days → candidate for removal
- Check via: Microsoft Graph `servicePrincipal` → `signInActivity`
- Also check: app registrations with no associated service principal usage
- Owner is a departed employee → reassign or decommission
- Disable before deleting — allow 30-day observation period

### Over-Permissioned Service Principals

Common over-permissions to flag:

| Permission | Risk | Typical Fix |
|-----------|------|-------------|
| `Directory.ReadWrite.All` | Can modify any directory object | Scope to specific operations |
| `RoleManagement.ReadWrite.Directory` | Can assign any role | Remove or scope |
| `Application.ReadWrite.All` | Can create/modify any app | Scope to owned apps |
| `Mail.ReadWrite` (application) | Read all users' mail | Scope to specific mailbox |
| Contributor at subscription | Full resource control | Reader or scoped role |

### Multi-Tenant App Registrations

- Review whether multi-tenant is actually needed
- Multi-tenant apps can be consented to by any Entra ID tenant
- If only used internally, switch to single-tenant
- Audit: who has consented, from which tenants

### Managed Identity Preferred

```
Priority order for workload authentication:

1. Managed identity (Azure workloads) — no credentials to manage
2. Workload identity federation (external workloads) — no secrets
3. Certificate credential — strong, harder to leak
4. Client secret — last resort, short-lived, automated rotation

Never: long-lived secrets, secrets in code, shared secrets across environments
```

### Hygiene Checklist

- [ ] All service principals have identifiable, current owners (at least 2)
- [ ] No credentials expiring beyond 1 year for certificates, 6 months for secrets
- [ ] Expiry alerts configured for all credentials
- [ ] Unused SPs (90+ days no sign-in) identified and disabled
- [ ] No application permissions with `*.ReadWrite.All` without justification
- [ ] Managed identity used for all Azure-hosted workloads
- [ ] Workload identity federation used for CI/CD pipelines
- [ ] Multi-tenant registrations reviewed for necessity
