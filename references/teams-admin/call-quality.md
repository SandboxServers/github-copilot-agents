# Teams Admin — Call Quality & Troubleshooting

## Call Quality & Troubleshooting

### Tools
| Tool | Purpose | Access |
|------|---------|--------|
| **Call Quality Dashboard (CQD)** | Org-wide call quality trends, network analysis, building mapping | Teams Admin Center → Analytics |
| **Call Analytics** | Per-user, per-call troubleshooting (media quality, network, client) | Teams Admin Center → Users → Call history |
| **Network Assessment Tool** | Pre-deployment network readiness testing | Downloadable tool |
| **Real-Time Telemetry** | Live meeting quality monitoring | Teams Admin Center (preview) |

### CQD Building Data
- Upload building/subnet mapping to correlate call quality with physical locations
- Identifies: which buildings have poor quality, which subnets are congested
- Format: CSV with subnet, building name, city, country, state, region

### Common Call Quality Issues
| Symptom | Likely Cause | Investigation |
|---------|-------------|--------------|
| Audio drops/freezes | Network packet loss > 5% | CQD → stream-level, check WiFi/VPN |
| Echo/feedback | Audio device issues, no AEC | Call Analytics → device tab |
| Robot voice | High jitter > 30ms | CQD → jitter metrics, check QoS |
| One-way audio | Firewall blocking media | Check UDP 49152-53247, media relay connectivity |
| Poor video quality | Bandwidth < 1.5 Mbps | CQD → video stream quality, bandwidth policy |
