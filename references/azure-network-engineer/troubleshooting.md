# Network Troubleshooting

## Network Watcher Tools

| Tool | Purpose |
|---|---|
| IP flow verify | Test if NSG allows/denies traffic — shows which rule applies |
| NSG diagnostics | Evaluate effective NSG rules for a NIC |
| Connection troubleshoot | Test TCP/ICMP connectivity from source to destination |
| Next hop | Determine effective route for traffic from a VM |
| Packet capture | Remote packet capture on VMs without SSH/RDP |
| VPN troubleshoot | Diagnose VPN gateway and connection issues |
| Connection Monitor | End-to-end connectivity monitoring between resources |
| Topology view | Visual map of network resources and connections |
| NSG flow logs | IP traffic logging (5-tuple: src/dst IP, port, protocol) |
| VNet flow logs | Preferred over NSG flow logs — simpler scope, less duplication |

## Troubleshooting Flow

Systematic approach for diagnosing network connectivity issues:

### Step 1: IP Flow Verify
- Is traffic allowed or denied by NSGs?
- Shows the specific NSG rule that matched
- Test from source VM to destination IP and port

### Step 2: Effective Routes
- What route table applies to the source NIC?
- Is there a UDR overriding the system route?
- Check for missing routes (e.g., no 0.0.0.0/0 → Firewall)

### Step 3: Connection Troubleshoot
- Can the source actually reach the destination on the target port?
- Tests TCP connectivity end-to-end
- Shows latency and reachability

### Step 4: NSG Flow Logs
- Review actual traffic flow history
- See allowed and denied flows over time
- Correlate with Traffic Analytics for visualization

### Step 5: Packet Capture
- Deep inspection of actual packets
- Use when flow logs don't provide enough detail
- Capture on source, destination, or both

## Common Issues and Fixes

### Asymmetric Routing
**Symptom**: Traffic enters via one path but returns via another. NVA/Firewall drops return traffic as it didn't see the initial connection.
**Fix**: Ensure UDRs route traffic symmetrically — both directions must traverse the same NVA/Firewall. Check route tables on all subnets involved.

### Missing UDR (Spoke-to-Spoke Failure)
**Symptom**: Spoke A can reach hub but cannot reach Spoke B.
**Fix**: Add UDR 0.0.0.0/0 → Firewall private IP on ALL spoke subnets. Ensure Firewall has rules allowing the spoke-to-spoke traffic. Verify IP forwarding is enabled on NVA.

### DNS Resolution Failure
**Symptom**: Private endpoint FQDN resolves to public IP instead of private IP.
**Fix**:
1. Check private DNS zone exists for the service (e.g., `privatelink.blob.core.windows.net`)
2. Check zone is linked to the VNet
3. Check A record exists with correct private IP
4. If custom DNS: verify conditional forwarder to 168.63.129.16

### NSG Blocking Traffic
**Symptom**: Connectivity fails even though firewall allows it.
**Fix**:
1. Check effective security rules on the NIC (combines subnet + NIC NSGs)
2. Remember default deny-all-inbound rule (priority 65500)
3. Verify both source subnet NSG and destination subnet NSG
4. Check if service tags are used correctly

### ExpressRoute Routing Issues
**Symptom**: Learned routes not propagating. On-premises can't reach Azure resources.
**Fix**:
1. Check BGP peering state (established vs idle)
2. Verify route filters — are prefixes being advertised?
3. Check route prefix count (4,000 limit, 10,000 with Premium)
4. Verify peering configuration on both sides

### VPN Gateway Issues
**Symptom**: S2S tunnel won't establish or drops intermittently.
**Fix**:
1. Run VPN troubleshoot in Network Watcher
2. Check gateway health metrics
3. Verify pre-shared key matches on both sides
4. Check IKE/IPsec parameters match (encryption, integrity, DH group)
5. Verify on-premises device configuration

## Traffic Analytics

- Built on NSG/VNet flow logs + Log Analytics workspace
- Visualizes: traffic patterns, hotspots, security threats, bandwidth utilization
- Identifies: open ports, talking hosts, blocked flows, geographic traffic sources
- Enable on all production VNets for ongoing visibility
- Use for capacity planning and security posture monitoring
