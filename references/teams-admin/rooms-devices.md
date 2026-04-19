# Teams Admin — Rooms & Devices

## Teams Rooms & Devices

### Device Categories
| Device | Purpose | Management |
|--------|---------|-----------|
| **Teams Rooms (Windows/Android)** | Conference room systems | Teams Rooms Pro license, TAC or Intune |
| **Teams Phones** | Desk phones, common area phones | Teams Admin Center, auto-provisioning |
| **Teams Displays** | Personal desk displays with Cortana | Teams Admin Center |
| **Teams Panels** | Room booking panels outside meeting rooms | Teams Admin Center |
| **SIP devices** | Legacy SIP phones with Teams integration | SIP Gateway |

### Resource Accounts
- Every Auto Attendant and Call Queue needs a resource account
- Resource accounts need: Teams Phone Resource Account license (free)
- Phone numbers assigned to resource accounts, not directly to AA/CQ
- Can be managed via Teams Admin Center or PowerShell

### Device Lifecycle
- **Provisioning**: Remote provisioning via Teams Admin Center, MAC address registration, or auto-discovery
- **Configuration profiles**: Apply settings (display, network, calling) to groups of devices
- **Firmware updates**: Managed via Teams Admin Center — auto-update, manual, or deferred
- **Monitoring**: Device health, call quality, offline alerts via Teams Admin Center → Devices
- **Retirement**: Factory reset, unassign, decommission from TAC
