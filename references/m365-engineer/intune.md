## Intune — Device Management

### Compliance Policies
```
Compliance Policy → Evaluates device → Compliant / Not Compliant → Conditional Access enforces

Common compliance requirements:
├── OS version minimum/maximum
├── Password requirements (length, complexity, expiration)
├── Encryption required (BitLocker on Windows, FileVault on macOS)
├── Not jailbroken/rooted
├── Threat level (integration with Defender for Endpoint)
└── Custom compliance (PowerShell scripts for advanced checks)

Non-compliance actions:
├── Mark device non-compliant (immediate or after grace period)
├── Send email notification to user
├── Remotely lock device
└── Retire device (remove corporate data)
```

### Configuration Profiles
| Profile Type | Use Case |
|-------------|----------|
| **Device restrictions** | Camera, Bluetooth, USB, screenshot policies |
| **Wi-Fi** | Corporate Wi-Fi SSID, certificate-based auth |
| **VPN** | Always-on VPN, per-app VPN |
| **Email** | Exchange Online profile, S/MIME |
| **Certificate** | SCEP/PKCS certificates for auth |
| **Endpoint protection** | BitLocker, FileVault, firewall, Defender settings |
| **Administrative templates (ADMX)** | Group Policy equivalent for Windows |
| **Settings catalog** | Granular settings from all supported CSPs |

### App Deployment
- **Microsoft 365 Apps**: deploy and manage Office suite, update channels (Current, Monthly Enterprise, Semi-Annual)
- **LOB apps**: .msi, .msix, .appx, .ipa, .apk — upload directly
- **Store apps**: Microsoft Store for Business, Apple VPP, managed Google Play
- **Win32 apps**: .intunewin format, detection rules, requirements, dependencies
- **App protection policies (MAM)**: protect corporate data in apps WITHOUT requiring device enrollment (BYOD)
