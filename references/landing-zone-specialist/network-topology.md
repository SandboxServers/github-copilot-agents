## Network Topology Decision Tree

```
Which network topology?
├─ Multi-region with global transit needed?
│  ├─ >30 branch sites or SD-WAN integration? → VWAN
│  ├─ Need VPN-to-ExpressRoute transitive routing? → VWAN
│  └─ Need Microsoft-managed hub with less ops overhead? → VWAN
├─ Single/few regions, full control needed?
│  ├─ Custom NVAs in hub? → Hub-Spoke (VWAN NVA support is limited)
│  ├─ Team has strong networking skills? → Hub-Spoke
│  └─ Need granular UDR control? → Hub-Spoke
└─ Greenfield, no strong preference?
   ├─ Small org, simple connectivity → Hub-Spoke (simpler to start)
   └─ Large org, multi-region from day one → VWAN
```

### Hub-Spoke vs VWAN Comparison

| Factor | Hub-Spoke (Traditional) | Virtual WAN |
|--------|------------------------|-------------|
| Management | Customer-managed hub VNet | Microsoft-managed service |
| NVA in hub | Full flexibility (any NVA) | Limited to supported NVAs |
| Spoke-to-spoke transit | Requires firewall/NVA routing | Built-in (any-to-any) |
| Multi-region connectivity | Manual hub peering + routing | Automatic hub-to-hub |
| Branch integration | VPN Gateway (≤30 tunnels practical) | Scales to 1000s of branches |
| SD-WAN integration | Manual | Native partner integration |
| ExpressRoute-to-VPN transit | Complex routing | Built-in |
| Operational overhead | Higher (you manage everything) | Lower (Microsoft manages routing) |
| Customization | Full control | Constrained to VWAN features |
| Cost | Pay for individual resources | VWAN hub hourly + scale units + data |
| UDR flexibility | Full control | Limited (improving) |
| Migration between | Possible but significant effort | Possible but significant effort |

### Network Decisions That Are Hard to Change Later

1. **IP address space** — Get this right. Overlapping ranges between on-prem and Azure cause endless pain. Plan for growth.
2. **DNS architecture** — Azure Private DNS zones, conditional forwarders, hybrid DNS resolution. Changing DNS after workloads are deployed is disruptive.
3. **ExpressRoute vs VPN** — ExpressRoute circuits have long lead times and contracts. Plan early.
4. **Private endpoint DNS strategy** — Centralized Private DNS zones in connectivity subscription vs distributed. Centralized is recommended.
5. **Hub-Spoke vs VWAN** — Migrating between them requires re-peering every spoke. Plan correctly.
