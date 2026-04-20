## Identity Integration

### Entra ID as Primary Identity Provider

Entra ID (formerly Azure AD) is the primary identity and access management service for Azure
Landing Zones. All user authentication, authorization, and conditional access policies originate
from Entra ID. Design identity architecture at the tenant level before deploying any landing
zone subscriptions.

### Hybrid Identity

Organizations with on-premises Active Directory require hybrid identity synchronization:

**Entra Connect** — traditional synchronization agent installed on-premises. Synchronizes users,
groups, and password hashes to Entra ID. Supports password hash sync, pass-through authentication,
or federation with AD FS.

**Entra Cloud Sync** — lightweight agent-based synchronization. Supports multi-forest topologies
and provides higher availability than Entra Connect. Preferred for new deployments.

Both synchronization options keep on-premises AD and Entra ID in sync so users authenticate with
the same credentials regardless of whether they access cloud or on-premises resources.

### Domain Controllers in Identity Subscription

When workloads require Active Directory Domain Services (AD DS) for Kerberos authentication, LDAP
queries, or Group Policy, deploy domain controllers in the **Identity subscription** under the
Platform management group. These VMs are peered to the hub and accessible from all landing zone
spokes. The platform identity team manages them — application teams consume the service.

Alternative: **Entra Domain Services** provides managed domain services without deploying VMs.
Suitable when legacy AD DS features (Group Policy authoring, schema extensions) are not required.

### Conditional Access at Tenant Level

Conditional Access policies apply to the entire tenant and control how users authenticate:
- Require MFA for all users (baseline policy)
- Block legacy authentication protocols
- Require compliant or hybrid-joined devices for sensitive applications
- Restrict access from untrusted locations for privileged roles
- Enforce phishing-resistant MFA (FIDO2, Windows Hello) for administrators
- Set sign-in frequency and session lifetime controls

Apply Conditional Access in layers: baseline for all users, stricter for privileged users,
and specific policies for guest/external access.

### Privileged Identity Management (PIM)

PIM provides just-in-time (JIT) privileged access:
- Eligible assignments require activation with justification and optional approval
- Time-bound activation windows (4-8 hours typical)
- Standing access only for roles that need continuous access (Reader for monitoring)
- Require MFA at activation time
- Alert on activation of highly privileged roles (Global Admin, Owner)
- Review eligible assignments quarterly to remove stale entries

### Break-Glass Accounts

Maintain two cloud-only emergency access accounts:
- Excluded from all Conditional Access policies
- Permanent Global Administrator assignment (not PIM-eligible)
- Complex passwords stored in a physical safe (not Key Vault — avoids circular dependency)
- No MFA configured (fallback when MFA infrastructure fails)
- Monitor and alert on any sign-in attempt
- Test quarterly to verify access works and rotate credentials

### Service Principal and Managed Identity Governance

Prefer **managed identities** over service principals — no credentials to manage or rotate.
Use **workload identity federation** for CI/CD pipelines (GitHub Actions, Azure DevOps) to
eliminate client secrets entirely. For service principals that require credentials, set expiration
to 6 months maximum, audit unused principals quarterly, and scope permissions to the minimum
necessary resource group or resource level.