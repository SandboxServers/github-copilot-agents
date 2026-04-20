# Entra ID Anti-Patterns

Common Entra ID misconfigurations and anti-patterns that weaken identity security posture.

---

## Over-Permissioned App Registrations

- Granting `Directory.ReadWrite.All` or `User.ReadWrite.All` when `User.Read` would suffice
- Requesting application-level Graph API permissions when delegated permissions meet the use case
- Using `*.All` permission scopes out of convenience — violates least-privilege principle
- **Fix:** Audit permissions with `Get-MgServicePrincipalOAuth2PermissionGrant` and scope down to exact operations needed

## No Conditional Access Baseline

- Tenants with no conditional access policies — MFA not enforced for any users
- Relying solely on per-user MFA (legacy) instead of conditional access-based MFA
- Not blocking legacy authentication protocols (IMAP, POP3, SMTP, ActiveSync with basic auth)
- **Fix:** Deploy Microsoft's conditional access baseline: require MFA for all users, block legacy auth, require compliant devices for sensitive apps

## Shared Service Principal Credentials

- Multiple teams sharing a single service principal with client secret passed via email or shared vault
- No accountability for which team performed which action — audit trail is useless
- **Fix:** One service principal per team per workload; use workload identity federation to eliminate secrets entirely

## Not Using Workload Identity Federation

- CI/CD pipelines (GitHub Actions, Azure DevOps) still authenticating with client secrets or certificates
- Secrets stored in pipeline variables — rotation burden, leak risk, expiration failures
- **Fix:** Configure federated identity credentials on app registrations; OIDC token exchange eliminates stored secrets

## No Break-Glass Accounts

- All admin accounts subject to conditional access and MFA — complete lockout if MFA provider fails
- Single break-glass account (need at least two, in case one is compromised)
- Break-glass accounts not excluded from conditional access policies
- **Fix:** Create 2+ cloud-only break-glass accounts, exclude from all CA policies, monitor sign-in logs with alerts

## Permanent Privileged Role Assignments

- Global Administrator permanently assigned to 10+ users
- No use of Privileged Identity Management (PIM) for just-in-time activation
- Service accounts with permanent Directory role assignments that are never reviewed
- **Fix:** Use PIM for all privileged roles; set maximum activation duration (8 hours); require justification and approval for Global Admin

## Legacy Authentication Protocols Still Enabled

- Basic authentication enabled on Exchange Online (IMAP, POP3, SMTP AUTH)
- Legacy protocols bypass conditional access MFA policies entirely
- Attackers use password spray against basic auth endpoints
- **Fix:** Block legacy auth via conditional access; disable basic auth at the tenant level via Exchange authentication policies

## Not Monitoring Sign-In Logs

- No alerting on risky sign-ins or risky users from Entra ID Protection
- Sign-in logs not exported to Log Analytics or SIEM
- Service principal sign-in logs ignored (separate log stream from user sign-ins)
- **Fix:** Stream sign-in and audit logs to Log Analytics; create alerts for impossible travel, risky sign-ins, and anomalous service principal activity

## Guest Users with Excessive Permissions

- Guest users defaulting to broad directory read permissions
- Not restricting guest invitation policies — any user can invite external guests
- Guests never reviewed or removed after project ends
- **Fix:** Restrict guest default permissions in External Collaboration settings; use access reviews for guest accounts quarterly

## App Registrations with Never-Expiring Secrets

- Client secrets created with no expiration or multi-year expiration
- No process to rotate secrets before expiry — leads to production outages
- Certificates with 10-year validity that nobody tracks
- **Fix:** Set maximum secret lifetime to 6 months via app management policy; automate rotation with Key Vault

## Custom RBAC Roles When Built-In Roles Suffice

- Creating custom Entra directory roles that duplicate built-in roles
- Custom roles with overly broad `microsoft.directory/*` action sets
- No documentation on why custom role was needed
- **Fix:** Always check built-in roles first; document justification for any custom role; review custom roles quarterly

## No Naming Convention for App Registrations

- App registrations named "Test App", "My App", "App1" — impossible to identify purpose
- No way to determine which team owns which registration
- Orphaned registrations accumulate because nobody knows if they're still needed
- **Fix:** Enforce naming convention: `{team}-{environment}-{purpose}` (e.g., `platform-prod-pipeline-auth`); require owners on all registrations

---

> **Sources:** [Microsoft identity platform best practices](https://learn.microsoft.com/en-us/entra/identity-platform/identity-platform-integration-checklist) · [Conditional access best practices](https://learn.microsoft.com/en-us/entra/identity/conditional-access/plan-conditional-access) · [Emergency access accounts](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-emergency-access)
