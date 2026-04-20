# Entra ID Best Practices

Proven practices for securing and operating Microsoft Entra ID tenants at scale.

---

## Conditional Access Baseline

- **Require MFA for all users** — start with a baseline policy targeting all cloud apps, all users (exclude break-glass)
- **Block legacy authentication** — create a policy blocking Exchange ActiveSync and other legacy clients; legacy auth cannot enforce MFA
- **Require device compliance** for access to sensitive applications (SharePoint, Exchange, Azure portal)
- **Require app protection policies** for mobile device access to corporate data
- Use **report-only mode** for 2 weeks before enforcing new policies; review sign-in logs for impact
- Layer policies: identity verification (MFA) → device trust (compliance) → app-level controls (session restrictions)

## Privileged Identity Management (PIM)

- Enable PIM for **all** privileged directory roles — especially Global Admin, Privileged Role Admin, Exchange Admin
- Set **maximum activation duration** to 8 hours; require justification on every activation
- Require **approval workflows** for Global Admin and Privileged Auth Admin activations
- Configure **activation alerts** — notify security team when high-impact roles are activated
- Set **eligible** assignments instead of permanent **active** assignments for all human users
- Service accounts that need permanent roles should be documented, monitored, and reviewed quarterly

## Workload Identity Federation

- Use **federated identity credentials** for all CI/CD pipelines — GitHub Actions, Azure DevOps, GitLab CI
- Eliminates client secrets and certificates from pipeline variable stores
- Configure subject claims tightly: lock to specific repos, branches, and environments
- For AKS workloads, use **workload identity** (successor to pod identity) — each pod gets its own federated identity
- For Terraform, use OIDC authentication with `use_oidc = true` in the AzureRM provider

## App Registration Hygiene

- **Naming convention:** `{team}-{env}-{purpose}` (e.g., `data-prod-etl-pipeline`)
- **Require owners:** Every app registration must have at least 2 owners; orphaned apps with no owner should be flagged
- **Credential expiration:** Set org-wide policy for maximum client secret lifetime (recommend 6 months)
- **Least-privilege permissions:** Request only the specific Graph scopes needed; avoid `*.ReadWrite.All` patterns
- **Regular audits:** Monthly report on app registrations with expiring credentials (30/60/90 day windows)
- Use app management policies to restrict who can create app registrations

## Break-Glass Accounts

- Create **at least 2** cloud-only break-glass accounts (no on-premises sync, no phone-based MFA)
- Use FIDO2 security keys or long complex passwords stored in a physical safe
- **Exclude from all conditional access policies** — these accounts must always be able to sign in
- Assign **permanent Global Admin** role (exception to PIM rule — must be always available)
- **Monitor actively:** Create an alert in Log Analytics that fires on any sign-in from break-glass accounts
- Test break-glass accounts quarterly — verify they can still sign in and perform admin tasks

## Access Reviews

- **Quarterly reviews** for all privileged role assignments (PIM eligible and active)
- **Annual reviews** for application access and group memberships
- **Guest access reviews** quarterly — auto-remove guests who don't respond within 14 days
- Assign review owners to resource owners, not central IT — the people closest to the resource know who should have access
- Enable **auto-apply** for reviews where non-response means removal (especially guest access)
- Review service principal role assignments separately — these are often forgotten

## Named Locations

- Define **trusted corporate networks** as named locations (office egress IPs, VPN exit points)
- Create separate named locations for **cloud provider egress** (Azure, AWS) used by CI/CD
- Use named locations in conditional access to reduce MFA friction for trusted networks
- Review named locations quarterly — IP ranges change as offices and VPNs evolve
- Mark trusted locations as **compliant network** locations when using Global Secure Access

## Managed Identity: Prefer Over Service Principals

- Use **managed identity** wherever the Azure resource supports it (App Service, Functions, AKS, VMs, Logic Apps)
- No credentials to manage, rotate, or leak — Azure handles the token lifecycle
- Prefer **user-assigned** managed identity for resources that may be recreated (scale sets, blue-green deployments)
- Use **system-assigned** for simpler scenarios where the identity lifecycle should match the resource
- Managed identities work with Azure RBAC, Key Vault access policies, and Storage data-plane roles

## Sign-In Monitoring and Entra ID Protection

- **Enable Entra ID Protection** — requires P2 license; provides risk-based conditional access
- Configure **risky sign-in policies:** require MFA for medium risk, block for high risk
- Configure **risky user policies:** require password change for high-risk users
- Export **all sign-in log categories** to Log Analytics: user sign-ins, service principal sign-ins, managed identity sign-ins
- Create alerts for: impossible travel, sign-ins from blocked countries, anomalous token activity
- Use **workbooks** in Entra for visual dashboards: sign-in success/failure trends, MFA registration status, CA policy hit rates
- Integrate with Microsoft Sentinel for cross-signal correlation (identity + endpoint + cloud apps)

---

> **Sources:** [Conditional access deployment plan](https://learn.microsoft.com/en-us/entra/identity/conditional-access/plan-conditional-access) · [PIM deployment plan](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-deployment-plan) · [Emergency access accounts](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-emergency-access) · [Workload identity federation](https://learn.microsoft.com/en-us/entra/workload-id/workload-identity-federation) · [Entra ID Protection](https://learn.microsoft.com/en-us/entra/id-protection/overview-identity-protection)
