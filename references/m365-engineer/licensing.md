## Licensing — What Requires What

### E3 vs E5 Feature Comparison (Key Differences)

| Feature | E3 | E5 |
|---------|----|----|
| **Exchange Online** | Plan 2 | Plan 2 |
| **SharePoint Online** | Plan 2 | Plan 2 |
| **Microsoft Teams** | Yes | Yes |
| **Microsoft 365 Apps** | Yes | Yes |
| **Intune Plan 1** | Yes | Yes |
| **Entra ID P1** | Yes | Yes |
| **Entra ID P2** | No | Yes |
| **Defender for Office 365** | No (Plan 1 basic) | Plan 2 (full) |
| **Defender for Endpoint** | Plan 1 | Plan 2 |
| **Defender for Identity** | No | Yes |
| **Defender for Cloud Apps** | No | Yes |
| **Purview DLP** | Exchange only | Exchange + Endpoint + Teams |
| **Auto-labeling (service-side)** | No | Yes |
| **Insider Risk Management** | No | Yes |
| **eDiscovery Premium** | No | Yes |
| **Communication Compliance** | No | Yes |
| **Advanced Audit** | No | Yes |
| **Phone System** | No | Yes (add calling plan separately) |
| **Power BI Pro** | No | Yes |

### When E5 Is Worth It
- **Security posture** requires full Defender suite (identity + endpoint + O365 + Cloud Apps)
- **Compliance** requires insider risk, advanced audit, eDiscovery Premium, communication compliance
- **Phone System** planned (E5 includes Phone System license, add calling plan)
- **Auto-labeling** at scale (service-side auto-labeling requires E5)
- **PIM** and risk-based conditional access (Entra ID P2 in E5)
- **Power BI Pro** needed org-wide (included in E5, $10/user/month add-on for E3)

### When E5 Is NOT Worth It
- Org doesn't need advanced compliance features
- Third-party security stack (Defender suite unused)
- Phone system not needed (or using third-party)
- Auto-labeling not required (manual labeling works)
- Better to add specific features à la carte than pay for full E5

### Common Add-On SKUs
| Add-On | Adds To | Key Feature |
|--------|---------|-------------|
| **E5 Security** | E3 | Full Defender suite + Entra P2 |
| **E5 Compliance** | E3 | Full Purview suite (DLP, insider risk, eDiscovery Premium) |
| **Audio Conferencing** | E3/E5 | Dial-in numbers for Teams meetings |
| **Calling Plan** | E5 | Phone numbers and PSTN minutes |
| **Teams Phone Standard** | E3 | Phone System for E3 users |
| **Intune Plan 2** | E3/E5 | Advanced endpoint management |
| **Intune Suite** | E3/E5 | Plan 2 + remote help, Tunnel, analytics |
| **SharePoint Advanced Management** | E3/E5 | Site lifecycle, access governance |
| **Multi-Geo** | E3/E5 | Data residency in multiple geos |
| **Copilot** | E3/E5 | AI assistant across M365 apps |
