## Zero Trust Identity for M365

### The Three Principles

Zero trust is not a product — it is the operating model for M365 security:

1. **Verify explicitly**: Evaluate every access request based on all available signals — user identity, device health, location, app, data sensitivity, real-time risk
2. **Use least privilege**: Grant minimum access needed, just-in-time, with time-bound elevation — PIM for admin roles, scoped RBAC, limited app permissions
3. **Assume breach**: Minimize blast radius, segment access, verify end-to-end encryption, use analytics to detect anomalies and drive response

### Conditional Access — The Unified Enforcement Point

Conditional Access is the policy engine that ties zero trust together. Every access decision flows through it:

| Signal | Source | What It Evaluates |
|--------|--------|-------------------|
| **User identity** | Entra ID | Who is requesting access — user, group membership, role |
| **Device compliance** | Intune | Is the device managed, compliant, encrypted, patched |
| **Identity risk** | Entra ID Protection | Sign-in risk (impossible travel, anonymous IP) and user risk (leaked credentials, anomalous behavior) |
| **Location** | Named locations | Trusted network vs untrusted vs impossible geography |
| **Application** | Target app | Which M365 service or third-party app is being accessed |
| **Session** | Defender for Cloud Apps | Real-time session controls — block download, enforce DLP in-session |

### The Device Trust Chain

Intune and Entra work as a feedback loop — this is the device enforcement chain:

1. **Intune defines compliance**: OS version requirements, encryption enabled, antivirus active, no jailbreak
2. **Conditional Access enforces compliance**: "Require compliant device" as a grant control
3. **App protection for unmanaged devices**: Intune app protection policies protect data on personal devices
4. **Conditional Access requires app protection**: Mobile access requires approved apps with protection policies
5. **Defender for Endpoint feeds risk**: Device risk signals flow into Intune compliance → Conditional Access

Break any link and the chain fails. A compliant device requirement without Intune enrollment policies means no devices can comply.

### Three-Tier Protection Model

| Tier | Audience | Key Controls |
|------|----------|-------------|
| **Starting point** | All users | MFA for all, block legacy authentication, basic Conditional Access, self-service password reset |
| **Enterprise** | Most users | Risk-based Conditional Access (sign-in risk, user risk), compliant device required, app protection policies, session lifetime controls |
| **Specialized security** | Executives, admins, sensitive roles | Managed device only, strict session controls, phishing-resistant MFA (FIDO2/passkeys), no persistent browser sessions |

### Eliminating Legacy Authentication

Legacy authentication protocols (IMAP, POP3, SMTP basic auth, older Office clients) do not support MFA. They are the primary bypass vector for Conditional Access:

- **Block legacy auth in Conditional Access**: Create a policy targeting "Other clients" with Block grant
- **Disable protocols at service level**: Exchange Online — disable POP, IMAP, SMTP AUTH per mailbox
- **Monitor sign-in logs**: Filter for "Legacy Authentication" client apps to find remaining usage
- **Remediate**: Identify apps and service accounts still using basic auth — migrate to modern auth or app passwords as a temporary bridge

### Privileged Identity Management (PIM)

Admin access should never be persistent:

- **Just-in-time activation**: Admins request role activation for a defined period (1-8 hours)
- **Approval workflows**: High-privilege roles require approval from another admin
- **Access reviews**: Quarterly review of who has standing access to privileged roles
- **Eligible vs active assignments**: Assign roles as eligible (must activate) not active (always on)
- **Scoped roles**: Use administrative units to scope admin permissions to specific user populations

### Monitoring and Anomaly Detection

Assume breach means continuous monitoring:

- **Entra ID Protection**: Automatic detection of risky sign-ins and risky users with auto-remediation (force password change, force MFA)
- **Continuous access evaluation (CAE)**: Near-real-time token revocation when conditions change (user disabled, IP changes, risk detected)
- **Defender for Identity**: On-premises Active Directory monitoring for lateral movement, pass-the-hash, reconnaissance
- **Unified audit log**: All identity events logged and searchable — integrate with Sentinel for advanced analytics
- **Alert policies**: Automated alerts for impossible travel, mass file downloads, role elevation patterns
