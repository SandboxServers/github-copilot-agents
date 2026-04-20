# Teams Admin — Teams Rooms & Devices

## Teams Rooms Devices

### Overview
- Dedicated meeting room systems running Microsoft Teams Rooms software
- Platforms: Windows-based (compute module + peripherals) or Android-based (all-in-one)
- Certified hardware from: Lenovo, HP, Poly, Yealink, Crestron, Logitech, Neat

### Teams Rooms on Windows
- Full Windows 10/11 IoT Enterprise with Teams Rooms app
- Supports: dual-screen, content camera, multiple microphones/speakers
- Management: Teams Admin Center, Intune, or Pro Management portal
- Requires: Teams Rooms Pro or Teams Rooms Basic license

### Teams Rooms on Android
- Android-based all-in-one devices (typically smaller rooms)
- Simpler deployment, integrated hardware
- Management: Teams Admin Center
- Requires: Teams Rooms Pro or Teams Rooms Basic license

## Licensing

### Teams Rooms Pro vs Basic
| Feature | Basic (Free, up to 25 rooms) | Pro |
|---------|------------------------------|-----|
| Join meetings | Yes | Yes |
| Wireless sharing | Yes | Yes |
| Pro Management portal | No | Yes |
| Remote device management | Limited | Full |
| AI-powered features | No | Yes (Intelligent speakers, voice recognition) |
| Digital signage | No | Yes |
| Custom backgrounds | No | Yes |

## Pro Management Portal

### Capabilities
- Centralized monitoring of all Teams Rooms devices across locations
- Automated incident detection and ticketing
- Firmware and app update management
- Room utilization analytics and health dashboards
- Requires Teams Rooms Pro license per device

## Room Accounts (Resource Accounts)

### Configuration
- Each Teams Room device signs in with a dedicated resource account
- Resource account: Exchange room mailbox + Teams Rooms license
- Calendar integration: room appears in Outlook room finder for booking
- Account requires: password (no MFA — device signs in non-interactively)
- Conditional access: exclude room accounts or use compliant device policy

## Other Teams Devices

### Surface Hub
- Large interactive whiteboard with integrated Teams experience
- Surface Hub 2S (50" and 85") and Surface Hub 3
- Supports: Teams meetings, whiteboarding, co-creation
- Managed via Intune or Teams Admin Center

### Teams Displays
- Personal desk devices with always-on Teams presence
- Shows: calendar, notifications, Teams status, hot-desking
- Supports: Cortana voice assistant, one-touch meeting join
- Requires: Teams Rooms Basic or Pro license (or Teams shared device license)

### Teams Panels
- Room scheduling panels mounted outside meeting rooms
- Shows: room availability, upcoming reservations, capacity
- Integrates with Exchange room mailbox for real-time booking
- Requires: Teams Rooms Basic or Pro license

### Teams Phones
- IP desk phones and common area phones certified for Teams
- Common area phones: shared devices in lobbies, break rooms, factory floors
- Common area phone license: reduced-cost license for shared scenarios
- Management: Teams Admin Center → Devices → Phones

## Device Management

### Configuration Profiles
- Apply standardized settings to groups of devices: display, network, time zone, calling
- Created in Teams Admin Center → Devices → Configuration profiles

### Firmware and Updates
- Managed via Teams Admin Center → Devices
- Options: auto-update, manual approval, deferred updates, phased rollout
- Firmware validation: Microsoft certifies firmware versions for each device model

### Monitoring and Alerts
- Device health dashboard: online/offline status, call quality, resource utilization
- Alerts: configurable notifications for device offline, peripheral disconnected, call quality degradation
- Integration with Pro Management portal for advanced monitoring (Pro license required)