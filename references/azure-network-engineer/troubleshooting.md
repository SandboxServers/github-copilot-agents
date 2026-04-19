# Troubleshooting

### Network Watcher Tools
- **Connection Monitor**: end-to-end connectivity monitoring between Azure resources
- **NSG flow logs**: IP traffic logging through NSGs (5-tuple: src/dst IP, src/dst port, protocol)
- **VNet flow logs**: preferred replacement for NSG flow logs — simpler scope, less duplication, supports encryption status
- **Packet capture**: remote packet capture on VMs without SSH/RDP
- **IP flow verify**: check if a packet is allowed/denied and which NSG rule applies
- **Next hop**: determine where traffic goes from a VM (effective routes)
- **VPN troubleshoot**: diagnose VPN gateway and connection issues
- **Connection troubleshoot**: test connectivity from VM to destination (TCP, ICMP)

### Traffic Analytics
- Built on NSG/VNet flow logs + Log Analytics workspace
- Visualizes: traffic patterns, hotspots, security threats, bandwidth utilization
- Identifies: open ports, talking hosts, blocked flows, geographic traffic sources
- **Best practice**: enable on all critical VNets and subnets

### Common Troubleshooting Patterns
1. **Traffic blocked unexpectedly**: IP flow verify → check effective NSG rules → check UDR effective routes
2. **DNS not resolving**: check private DNS zone links → check A records → check DNS server settings on VNet
3. **Private endpoint not reachable**: verify DNS resolution returns private IP → check NSG on source subnet → check VNet peering
4. **VPN connectivity issue**: VPN troubleshoot → check gateway health → verify BGP status → check on-prem device config
5. **Asymmetric routing**: check route tables on both sides → verify ExpressRoute/VPN route preference → check NAT rules
