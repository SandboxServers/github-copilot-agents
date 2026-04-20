## M365 Security

### Defender for Office 365

| Feature | Plan 1 | Plan 2 |
|---------|--------|--------|
| Safe Attachments | Yes | Yes |
| Safe Links | Yes | Yes |
| Anti-phishing policies | Basic | Advanced (impersonation, mailbox intelligence) |
| Real-time reports | Yes | Yes |
| Threat Explorer | No | Yes (full) |
| Automated investigation and response | No | Yes |
| Attack simulation training | No | Yes |

- **Safe Attachments**: detonates attachments in sandbox, blocks malicious files
- **Safe Links**: rewrites URLs, checks at time-of-click, blocks malicious destinations
- **Anti-phishing**: detects impersonation of users and domains, mailbox intelligence for contact patterns

### Defender for Endpoint Integration
- Endpoint detection and response (EDR) — investigate and remediate threats on devices
- Threat and vulnerability management — identify unpatched software, misconfigurations
- Attack surface reduction (ASR) rules — block common malware techniques
- Integrates with Intune compliance — device risk level feeds into Conditional Access
- Automated investigation and response for endpoint alerts

### Attack Simulation Training
- Phishing simulations: credential harvest, link in attachment, drive-by URL, OAuth consent
- Training modules assigned automatically to users who fail simulations
- Campaigns: track completion rates, repeat offenders, improvement over time
- Requires Defender for Office 365 Plan 2 or E5

### Security Defaults vs Conditional Access

| Feature | Security Defaults | Conditional Access |
|---------|-------------------|-------------------|
| MFA enforcement | All users | Granular — by user, app, risk, location |
| Legacy auth blocking | Yes | Configurable per policy |
| Customization | None | Full (conditions, grants, session controls) |
| License required | Free | Entra ID P1 minimum |
| Recommendation | Small orgs, no P1 | All orgs with P1 or higher |

### MFA Enforcement
- **Security defaults**: require MFA for all users (simple, no configuration)
- **Conditional Access**: require MFA based on risk, location, device, app sensitivity
- **Per-user MFA**: legacy approach — avoid, use Conditional Access instead
- **Authentication methods**: Microsoft Authenticator (preferred), FIDO2, Windows Hello, phone, SMS (least secure)
- **Number matching**: enabled by default — prevents MFA fatigue attacks

### Legacy Authentication Blocking
- Legacy protocols (POP, IMAP, SMTP AUTH, EWS basic auth) do not support MFA
- Block via Conditional Access: condition = client apps → legacy authentication clients → block
- Monitor sign-in logs for legacy auth usage before blocking
- Exceptions: may need SMTP AUTH for multifunction printers or LOB apps (use per-user exception)

### Secure Score
- Measures security posture across identity, devices, apps, data, infrastructure
- Recommendations prioritized by impact and implementation difficulty
- Track improvement over time, compare against similar organizations
- Actions: enable MFA, disable legacy auth, configure DLP, enable audit logging
- Not a compliance certification — a guidance and tracking tool

### Threat Explorer
- Real-time and historical view of email threats (phishing, malware, campaigns)
- Investigate: filter by sender, recipient, subject, delivery action, detection technology
- Actions: soft delete, hard delete, move to junk, trigger investigation
- Campaign views: group related threats, identify targeted users
- Requires Defender for Office 365 Plan 2

### Quarantine Management
- Quarantined messages: admin review, user review (configurable per policy)
- Quarantine policies: control what users can do (preview, release, request release, delete)
- Admin quarantine: global admin or security admin release or delete
- Notification: quarantine digest emails for end users (configurable frequency)
- Retention: quarantined items retained for 30 days by default
