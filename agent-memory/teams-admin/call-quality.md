# Teams Admin — Call Quality & Troubleshooting

## Call Quality Dashboard (CQD)

### Purpose
- Org-wide view of call and meeting quality trends across the entire tenant
- Analyze quality by location, network, device type, client version
- Identify systemic issues affecting groups of users or sites

### Building and Subnet Mapping
- Upload building/subnet data to correlate call quality with physical locations
- CSV format: subnet, building name, city, country, state, region, network name
- Enables reports like "Building X has 15% poor quality streams on WiFi"
- Without building data, CQD shows IP addresses only — no location context

### Key Reports
- Overall quality: percentage of poor streams by audio, video, screen sharing
- Location-enhanced reports: quality breakdown by building, network, subnet
- Client/device reports: quality by Teams client version, OS, audio device
- Server-client vs client-client: identify if issue is network or endpoint

## Call Analytics (Per-User)

### Purpose
- Per-user, per-call troubleshooting for individual quality issues
- Access: Teams Admin Center → Users → select user → Call history

### Diagnostic Information
- Media quality metrics: jitter, packet loss, round-trip time per stream
- Network details: local/remote IP, subnet, transport relay, WiFi/wired
- Device information: audio device, driver version, capture/render device
- Client information: Teams version, OS, platform

### Role-Based Access
- Teams admin: full access to all users' call analytics
- Teams communications support engineer: full call analytics access
- Teams communications support specialist: limited view (no caller identity on far end)

## Quality of Service (QoS)

### DSCP Marking
- Differentiated Services Code Point values mark packet priority
- Audio: DSCP 46 (EF — Expedited Forwarding), source port range 50000-50019
- Video: DSCP 34 (AF41), source port range 50020-50039
- Screen sharing: DSCP 18 (AF21), source port range 50040-50059

### Implementation
- Client-side: Group Policy (Windows), Intune configuration profiles, or MDM
- Network-side: configure network switches and routers to honor DSCP markings
- Port-based QoS: set client media port ranges in Teams Admin Center → Meetings → Meeting settings
- QoS only effective when network infrastructure is configured end-to-end

## Network Planning

### Bandwidth Requirements Per Modality
| Modality | Minimum | Recommended |
|----------|---------|-------------|
| Audio (1:1) | 30 Kbps | 70 Kbps |
| Video (1:1, 720p) | 1.5 Mbps | 2.5 Mbps |
| Screen sharing | 250 Kbps | 2 Mbps |
| Large meeting (video gallery) | 2.5 Mbps | 4 Mbps |

### Network Requirements
- UDP ports 3478-3481 to Microsoft transport relays (TURN/STUN)
- Media port range: UDP 50000-50059 (when QoS configured)
- Low latency path to Microsoft 365 endpoints (avoid hairpinning through VPN)
- Microsoft Network Assessment Tool: validates UDP connectivity, jitter, packet loss, latency

## Common Issues and Troubleshooting

### Symptom → Cause → Investigation
| Symptom | Likely Cause | Action |
|---------|-------------|--------|
| Audio drops/freezes | Packet loss > 5% | CQD stream-level analysis; check WiFi, VPN, network congestion |
| Echo or feedback | Audio device issue, no AEC | Call Analytics → device tab; check certified headset |
| Robot/garbled voice | High jitter > 30ms | CQD jitter metrics; verify QoS configuration |
| One-way audio | Firewall blocking media | Verify UDP 3478-3481 open; check media relay connectivity |
| Poor video quality | Bandwidth < 1.5 Mbps | CQD video stream quality; check bandwidth policies |
| Delayed audio | High round-trip time > 200ms | Check network path; avoid VPN for Teams media traffic |

### Troubleshooting Flow
1. User reports issue → check Call Analytics for that specific call
2. Identify: was it network (jitter/loss), device, or client issue?
3. If systemic (multiple users) → check CQD for location/subnet patterns
4. Correlate with building data → identify physical location
5. Engage network team if infrastructure issue; endpoint team if device issue
6. For real-time issues: use Real-Time Telemetry (preview) in Teams Admin Center
