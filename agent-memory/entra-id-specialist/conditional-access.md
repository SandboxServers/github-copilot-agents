## Conditional Access

### Signal-Based Access Control

Conditional Access evaluates signals at sign-in time and enforces access controls:

```
IF (signals match conditions)
  THEN (enforce controls — grant, block, or session restriction)
```

### Signals (Conditions)

| Signal | Examples |
|--------|----------|
| **User or group** | Specific users, groups, roles, guests |
| **Location** | Named locations, trusted IPs, country/region |
| **Device state** | Compliant, hybrid Entra joined, managed, unmanaged |
| **Client app** | Browser, mobile app, desktop client, legacy auth |
| **Application** | Specific cloud apps or all apps |
| **Sign-in risk** | Real-time risk level from Entra ID Protection |
| **User risk** | Aggregate risk level from user behavior patterns |
| **Authentication context** | Step-up for sensitive operations (C1, C2, C3) |

### Access Controls

**Grant controls**:
- Require MFA
- Require compliant device
- Require hybrid Entra joined device
- Require approved client app
- Require app protection policy
- Require authentication strength (passwordless, phishing-resistant)
- Require password change
- Block access

**Session controls**:
- Sign-in frequency (how often re-auth is required)
- Persistent browser session (remember me)
- Conditional Access App Control (Defender for Cloud Apps proxy)
- Continuous access evaluation (near real-time token revocation)

### Policy Layering Strategy

```
Layer 1 — Baseline (All Users):
├── Block legacy authentication
├── Require MFA for all users
└── Block sign-in from restricted countries

Layer 2 — Privileged Users (Admins):
├── Require phishing-resistant MFA (FIDO2, Windows Hello)
├── Require compliant device
├── Block from untrusted locations
└── Sign-in frequency: every session

Layer 3 — Application-Specific:
├── Sensitive apps: compliant device + MFA
├── Guest access: MFA required, block non-compliant devices
└── DLP-sensitive apps: Conditional Access App Control

Layer 4 — Risk-Based:
├── High user risk: force password change + MFA
├── High sign-in risk: require MFA
└── Medium sign-in risk: require MFA
```

### Named Locations

- **Trusted IPs**: corporate office ranges, VPN egress points
- **Country/region**: allow or block by geography
- Mark trusted locations to skip MFA when on corporate network
- GPS-based locations (requires Authenticator app)
- Use for: location-based policies, risk reduction, compliance

### Break-Glass Exclusions

- Two cloud-only Global Admin accounts excluded from ALL CA policies
- Not assigned to any individual — shared emergency accounts
- No MFA (or FIDO2 key in safe) — must work when MFA is down
- Strong passwords (40+ characters), stored securely
- Monitor every sign-in with alerts (Sentinel, Azure Monitor)
- Test quarterly to verify they still work

### What-If Tool

- Test policies before enforcement: simulate a sign-in scenario
- Inputs: user, app, location, device, client app, risk level
- Output: which policies apply, what controls are enforced
- Use before every policy change to catch unintended lockouts

### Report-Only Mode

- Deploy new policies in report-only first (no enforcement)
- Monitor sign-in logs for policy impact
- Check: who would be blocked, who would need MFA
- Convert to enabled only after validating expected behavior
- Recommended duration: 1-2 weeks in report-only before enabling

### Common Lockout Scenarios to Prevent

1. MFA service outage — break-glass accounts bypass MFA
2. Blocking legacy auth without checking sign-in logs first
3. Requiring compliant device for all apps — blocks device enrollment
4. No break-glass exclusion in new policy — admins locked out
5. Blocking all countries except HQ — traveling employees blocked
