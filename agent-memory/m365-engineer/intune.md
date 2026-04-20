## Intune — Device Management

### Device Enrollment

| Platform | Method | Description |
|----------|--------|-------------|
| Windows | Autopilot | Zero-touch provisioning, user-driven or self-deploying, pre-registered hardware hashes |
| Windows | Entra ID Join + Auto-enrollment | User joins device to Entra ID, MDM enrollment triggers automatically |
| Windows | GPO-based co-management | SCCM + Intune co-management for hybrid environments |
| Apple (iOS/iPadOS) | ADE (Automated Device Enrollment) | Formerly DEP, supervised mode, zero-touch, corporate-owned |
| Apple (macOS) | ADE | Automated enrollment during Setup Assistant |
| Apple | User enrollment | BYOD, managed Apple ID, work profile isolation |
| Android | Android Enterprise — fully managed | Corporate-owned, full device management |
| Android | Android Enterprise — work profile | BYOD or corporate-owned, personal/work separation |
| Android | Android Enterprise — dedicated | Kiosk and shared device scenarios |

### Compliance Policies
```
Compliance Policy → Evaluates device → Compliant / Not Compliant → Conditional Access enforces

Common compliance requirements:
├── OS version minimum and maximum
├── Password requirements (length, complexity, expiration)
├── Encryption required (BitLocker, FileVault)
├── Not jailbroken or rooted
├── Threat level (Defender for Endpoint integration)
├── Firewall enabled
└── Custom compliance (PowerShell scripts for advanced checks)

Non-compliance actions:
├── Mark device non-compliant (immediate or after grace period)
├── Send email notification to user
├── Remotely lock device
├── Retire device (remove corporate data)
└── Push notification to Company Portal
```

### Configuration Profiles

| Profile Type | Use Case |
|-------------|----------|
| Device restrictions | Camera, Bluetooth, USB, screenshot policies |
| Wi-Fi | Corporate SSID, certificate-based authentication |
| VPN | Always-on VPN, per-app VPN, split tunneling |
| Email | Exchange Online profile, S/MIME configuration |
| Certificates | SCEP and PKCS certificates for authentication |
| Endpoint protection | BitLocker, FileVault, firewall, Defender settings |
| Security baselines | Pre-configured security settings based on Microsoft recommendations |
| Administrative templates (ADMX) | Group Policy equivalent for Windows |
| Settings catalog | Granular settings from all supported CSPs |

### App Management

| Assignment Type | Behavior |
|----------------|----------|
| Required | Automatically installed, user cannot remove |
| Available | Published to Company Portal, user chooses to install |
| Uninstall | Removes the app from targeted devices |

- **Microsoft 365 Apps**: deploy Office suite, manage update channels (Current, Monthly Enterprise, Semi-Annual)
- **LOB apps**: .msi, .msix, .appx, .ipa, .apk — direct upload
- **Store apps**: Microsoft Store, Apple VPP, managed Google Play
- **Win32 apps**: .intunewin format, detection rules, requirements, dependencies, supersedence

### App Protection Policies (MAM Without Enrollment)
- Protect corporate data in apps on unmanaged BYOD devices
- Controls: prevent copy/paste to personal apps, require PIN, encrypt app data
- Selective wipe: remove only corporate data, leave personal data intact
- Supported apps: Outlook, Teams, Edge, OneDrive, Office mobile apps, third-party MAM-enabled apps
- No device enrollment required — policy applied at app level

### Conditional Access Integration
- Intune compliance status feeds into Entra Conditional Access
- Grant access only when device is marked compliant
- Combine with: user risk, sign-in risk, location, app sensitivity
- Require app protection policy for mobile app access (MAM CA)
- Device compliance alone does not block access — Conditional Access is the enforcement layer
