# Identity Security

Identity is the primary security perimeter in cloud. Every review starts here.

## Zero Trust Principles

All identity decisions follow zero trust:

- **Verify explicitly** — always authenticate and authorize based on all available data points (user identity, location, device health, service or workload, data classification, anomalies)
- **Use least privilege** — limit user access with just-in-time and just-enough-access (JIT/JEA), risk-based adaptive policies, and data protection
- **Assume breach** — minimize blast radius and segment access, verify end-to-end encryption, use analytics for threat detection and defense improvement

## Managed Identity

Managed identity is the preferred authentication method. No secrets to manage, rotate, or leak.

- **System-assigned** — tied to the resource lifecycle. When the resource is deleted, the identity is deleted. Use for single-resource scenarios (one App Service accessing one Key Vault).
- **User-assigned** — independent lifecycle, can be shared across multiple resources. Use when multiple resources need the same permissions or when identity needs to survive resource recreation.
- **Always prefer managed identity over service principals with secrets.** Every connection string is a secret that can leak. Managed identity eliminates this risk.
- **Grant minimal RBAC** — Storage Blob Data Reader not Contributor, Key Vault Secrets User not Key Vault Administrator

## RBAC Best Practices

- **Use built-in roles** — they're well-scoped and maintained by Microsoft. Custom roles only when no built-in fits.
- **Assign at narrowest scope** — resource group or resource level, not subscription. Subscription-level assignments are a finding.
- **No Owner at subscription level for workloads** — Owner should be reserved for platform team. Workload teams get Contributor at resource group level maximum.
- **Use PIM for just-in-time elevation** — no permanent privileged assignments. Require justification and approval for activation. Set 8-hour maximum activation.
- **Review assignments quarterly** — use Entra ID access reviews. Remove stale assignments. Verify each assignment is still needed.
- **Deny assignments** — use Azure deny assignments or custom roles with NotActions for guardrails

## Service Principal Hygiene

- **Certificate credentials over client secrets** — certificates are harder to exfiltrate and can be stored in Key Vault
- **Short expiry** — 6 months maximum for secrets, 12 months for certificates. Automate rotation.
- **Workload identity federation** — eliminate secrets entirely for GitHub Actions, AKS pods, external identity providers. Preferred over certificates.
- **Monitor with Entra ID sign-in logs** — detect unusual sign-in patterns, failed authentications, sign-ins from unexpected locations
- **No multi-tenant app registrations unless business requires it** — single-tenant by default, multi-tenant is a security expansion
- **Audit quarterly** — identify stale service principals, unused app registrations, over-permissioned assignments

## Conditional Access

- **Require MFA for all users** — no exceptions except break-glass accounts. Start with all cloud apps.
- **Block legacy authentication** — legacy protocols (POP, IMAP, SMTP AUTH) don't support MFA. Block them.
- **Require compliant devices** — for sensitive applications, require Intune-compliant or Hybrid Azure AD joined devices
- **Location-based policies** — block or require additional verification from unexpected countries. Named locations for trusted networks.
- **Sign-in risk policies** — with Entra ID P2, block high-risk sign-ins, require MFA for medium risk. Investigate flagged sign-ins.
- **Session controls** — enforce sign-in frequency, persistent browser controls, app-enforced restrictions for unmanaged devices

## Break-Glass Accounts

- **Exactly 2 cloud-only global admin accounts** — no on-prem sync, no MFA (since they bypass conditional access)
- **Excluded from ALL conditional access policies** — this is their purpose, they're the fallback when CA locks everyone out
- **Hardware FIDO2 security keys** — stored in a physical safe, split custody (two people needed)
- **Monitor every sign-in with alerts** — any break-glass sign-in triggers immediate investigation
- **Test quarterly** — verify the accounts work, credentials are valid, alerts fire correctly
- **Document the process** — when to use, who approves, what to do after use

## Anti-Patterns (Always a Finding)

- Contributor or Owner at subscription level for workload teams (too broad)
- Shared service principal used by multiple teams (no accountability)
- Client secrets in app settings or environment variables (use managed identity)
- No PIM configured (all privileged access is permanent)
- No conditional access policies (no MFA enforcement)
- Permanent Global Admin assignments (use PIM with justification)
- App registrations with no expiry on credentials (secrets never rotate)
- No access reviews configured (stale permissions accumulate indefinitely)
