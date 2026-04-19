## Conditional Access — Deep Design

### Policy Architecture
```
Condition (WHO + WHAT + WHERE + WHEN + RISK)
  → Control (GRANT or BLOCK or SESSION)

Assignments (Conditions):
├── Users and groups (include/exclude)
├── Cloud apps or actions (include/exclude)
├── Conditions:
│   ├── Sign-in risk level (high, medium, low, none)
│   ├── User risk level (high, medium, low, none)
│   ├── Device platforms (iOS, Android, Windows, macOS, Linux)
│   ├── Locations (named locations, trusted IPs, countries)
│   ├── Client apps (browser, mobile apps, desktop clients, legacy auth)
│   ├── Device state (compliant, hybrid joined, managed)
│   └── Authentication context (step-up for sensitive operations)

Access Controls:
├── Grant:
│   ├── Require MFA
│   ├── Require compliant device
│   ├── Require Entra hybrid joined device
│   ├── Require approved client app
│   ├── Require app protection policy
│   ├── Require authentication strength (passwordless, phishing-resistant)
│   ├── Require password change
│   └── Require terms of use
├── Block access
└── Session:
    ├── App enforced restrictions
    ├── Conditional Access App Control (Defender for Cloud Apps)
    ├── Sign-in frequency
    ├── Persistent browser session
    └── Customize continuous access evaluation
```

### Policy Layering Strategy
```
Layer 1: Baseline (All Users)
├── Block legacy authentication (client apps = other clients)
├── Require MFA for all users (with trusted location exclusion if needed)
└── Block sign-in from blocked countries

Layer 2: Privileged Users (Admins)
├── Require phishing-resistant MFA (FIDO2 or Windows Hello)
├── Require compliant device
├── Block access from untrusted locations
└── Sign-in frequency: every session

Layer 3: Application-Specific
├── Sensitive apps: require compliant device + MFA
├── Guest access: require MFA, block from non-compliant devices
└── Conditional Access App Control for DLP-sensitive apps

Layer 4: Risk-Based
├── High user risk: force password change + MFA
├── High sign-in risk: require MFA
└── Medium sign-in risk: require MFA
```

### Break-Glass Accounts
- **Two cloud-only Global Admin accounts** — permanently assigned, not managed by PIM
- **NOT assigned to any individual** — shared emergency accounts
- **Excluded from ALL conditional access policies** (explicit exclusion in every policy)
- **No MFA** (or use FIDO2 key stored in safe) — must work when MFA infrastructure is down
- **Strong passwords** (40+ characters, stored securely, split between safes/people)
- **Monitored**: alert on ANY sign-in to break-glass accounts (Azure Monitor, Sentinel)
- **Tested regularly**: quarterly sign-in test to verify they work
- **Naming**: avoid obvious names like "BreakGlass" — use non-descriptive names

### Common Lockout Scenarios to Prevent
1. **MFA service outage** → break-glass accounts bypass MFA
2. **Blocking all legacy auth without checking** → Exchange ActiveSync, older Outlook clients break
3. **Requiring compliant device for all apps** → users can't enroll their device (chicken-and-egg)
4. **No exclusion for break-glass in new policy** → admins locked out
5. **Blocking all countries except HQ** → traveling employees, remote workers blocked
