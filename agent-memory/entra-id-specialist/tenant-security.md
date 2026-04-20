## Tenant Security Hardening

### Security Baseline

Every Entra ID tenant should meet these minimums:

| Control | Minimum | Recommended |
|---------|---------|-------------|
| **MFA** | Security defaults | Conditional Access with auth strength |
| **Legacy auth** | Not blocked (defaults) | Blocked via Conditional Access policy |
| **Admin protection** | Basic MFA | Phishing-resistant MFA + PIM + compliant device |
| **Guest access** | Default permissions | Scoped access, regular reviews |
| **Password reset** | Admin-only | Self-service password reset (SSPR) enabled |

### Security Defaults vs Conditional Access

**Security defaults** (minimum — free tier):
- MFA required for all users (Microsoft Authenticator)
- MFA required for admin roles
- Legacy authentication blocked
- Good starting point but limited customization

**Conditional Access** (recommended — requires P1):
- Granular signal-based policies
- Authentication strength (phishing-resistant)
- Device compliance requirements
- Risk-based policies
- Turn OFF security defaults when using CA

### Legacy Authentication

- Block via Conditional Access: Client apps → Exchange ActiveSync + Other clients
- Check sign-in logs before blocking — identify dependent systems
- Common dependencies: printers, scanners, older mail clients
- Migration path: modern auth clients, OAuth for SMTP where supported

### MFA Enforcement

- Registration campaign: require MFA registration for all users
- Nudge users to register Authenticator app
- Phishing-resistant methods for admins: FIDO2, Windows Hello, certificate-based
- External authentication methods for third-party MFA integration

### Admin Account Security

- Admin accounts should be cloud-only (not synced from on-prem AD)
- Separate admin accounts from daily-use accounts
- No mail-enabled admin accounts (reduces phishing attack surface)
- All admin roles managed through PIM (eligible, not permanent)

### Global Admin Minimization

- Target: fewer than 5 Global Admins (2 should be break-glass only)
- Use least-privilege admin roles instead:
  - User Administrator, Groups Administrator, Authentication Administrator
  - Exchange Admin, SharePoint Admin, Teams Admin
  - Security Reader, Compliance Administrator
- Audit current Global Admins — most can be scoped to specific roles

### Emergency Access Accounts

- **Two** cloud-only break-glass accounts
- Excluded from ALL Conditional Access policies
- Not managed by PIM (permanent active Global Admin)
- Strong passwords (40+ characters) stored in separate secure locations
- No MFA or FIDO2 key in physical safe
- Tested quarterly: sign in, verify access, document test
- Monitored: alert on ANY sign-in activity

### Entra ID Protection

**User risk policies**:
- High risk: require password change + MFA
- Medium risk: require MFA
- Sources: leaked credentials, impossible travel, anonymous IP

**Sign-in risk policies**:
- High risk: block or require MFA
- Medium risk: require MFA
- Sources: unfamiliar location, malware-linked IP, atypical travel

### Audit and Sign-In Logs

- **Sign-in logs**: 30 days retained by default in Entra ID
- **Audit logs**: 30 days retained by default
- **Export to**: Log Analytics workspace for long-term retention (recommended: 1+ year)
- **Route to**: Microsoft Sentinel for security analysis and alerting
- Key queries: failed sign-ins, risky sign-ins, service principal activity, role changes
- Enable diagnostic settings on the Entra ID tenant

### Hardening Checklist

- [ ] Conditional Access replaces security defaults
- [ ] Legacy authentication blocked
- [ ] MFA required for all users
- [ ] Phishing-resistant MFA for all admin roles
- [ ] Admin accounts cloud-only, no mailbox
- [ ] Global Admin count < 5 (including break-glass)
- [ ] Break-glass accounts: 2, excluded from CA, monitored, tested quarterly
- [ ] PIM enabled for all admin roles
- [ ] SSPR enabled for all users
- [ ] Entra ID Protection: user risk + sign-in risk policies enabled
- [ ] Logs exported to Log Analytics / Sentinel for retention and alerting
