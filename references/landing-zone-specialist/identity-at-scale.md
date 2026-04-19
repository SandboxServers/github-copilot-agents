## Identity at Scale

### PIM (Privileged Identity Management) Design

```
Standing Access (permanent assignments):
├─ Reader on Landing Zones MG → Platform team (always need visibility)
├─ Network Operations (custom) → Network team (routine tasks)
└─ Security Operations (custom) → Security team (monitoring)

Eligible Access (JIT activation via PIM):
├─ Owner on specific subscription → App team leads (when needed)
├─ Contributor on platform subscriptions → Platform admins (4-hour max)
├─ Global Administrator → Emergency only (8-hour max, approval required)
└─ User Access Administrator → RBAC changes (requires justification)
```

### Break-Glass Accounts

- **Two cloud-only accounts** (not synced from on-prem)
- **Excluded from ALL conditional access policies**
- **Complex passwords stored in physical safe** (not in Key Vault — that creates circular dependency)
- **No MFA** (by design — they're the fallback when MFA is broken)
- **Monitored**: Alert on ANY sign-in attempt (should never be used in normal operations)
- **Tested quarterly**: Actually sign in, verify access works, rotate credentials
- **Assigned Global Administrator permanently** (not PIM-eligible — they're the last resort)

### Service Principal Hygiene

| Practice | Why |
|----------|-----|
| Use managed identity instead of service principals where possible | No credentials to manage or rotate |
| Use workload identity federation for CI/CD | No client secrets needed for GitHub Actions / Azure DevOps |
| Set credential expiration ≤ 6 months | Limits blast radius of leaked credentials |
| Audit unused service principals quarterly | Orphaned SPs with permissions are attack vectors |
| Scope to minimum necessary | Don't give Contributor on subscription when Contributor on RG suffices |
| Name with purpose: `sp-<app>-<env>-<purpose>` | Makes audit and lifecycle management possible |

### Conditional Access Policy Layering

```
Layer 1: Baseline (all users)
├─ Require MFA for all users
├─ Block legacy authentication
├─ Require compliant device OR MFA
└─ Session: sign-in frequency 24 hours

Layer 2: Privileged Users
├─ Require phishing-resistant MFA (FIDO2/Windows Hello)
├─ Require compliant device (no BYOD)
├─ Block access from non-trusted locations
└─ Session: sign-in frequency 4 hours

Layer 3: Protected Actions (Entra ID admin actions)
├─ Require phishing-resistant MFA for role assignments
├─ Require phishing-resistant MFA for conditional access changes
└─ Require phishing-resistant MFA for cross-tenant access changes

Layer 4: Guest/External
├─ Require MFA (accept federated MFA claims)
├─ Block access to sensitive apps
└─ Session: sign-in frequency 8 hours
```
