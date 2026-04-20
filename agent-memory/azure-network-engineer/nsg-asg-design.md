# NSG and ASG Design

## Network Security Groups (NSGs)

Stateful Layer 3/Layer 4 firewall applied at the subnet or NIC level. Rules filter traffic by source/destination IP, port, and protocol.

### How NSGs Work
- Associate to subnet and/or NIC — both can be applied simultaneously
- Inbound evaluation order: subnet NSG first, then NIC NSG
- Outbound evaluation order: NIC NSG first, then subnet NSG
- Rules processed by priority (100–4096) — lower number = higher priority, first match wins
- Leave gaps in priority numbers (100, 200, 300) for future rule insertions

### Default Rules (Cannot Delete, Can Override)
- AllowVNetInBound (priority 65000) — allows all intra-VNet traffic
- AllowAzureLoadBalancerInBound (priority 65001) — allows health probes
- DenyAllInBound (priority 65500) — denies everything else inbound
- AllowVNetOutBound (priority 65000) — allows all outbound to VNet
- AllowInternetOutBound (priority 65001) — allows outbound to Internet
- DenyAllOutBound (priority 65500) — denies everything else outbound

### Service Tags
- Built-in tags: `VirtualNetwork`, `Internet`, `AzureLoadBalancer`, `AzureCloud`, `Storage`, `Sql`, `AzureMonitor`, `AzureKeyVault`, etc.
- Use service tags instead of IP ranges — automatically updated by Microsoft
- **Gotcha**: `VirtualNetwork` tag includes VNet address space + peered VNets + on-premises addresses via gateways — not just local VNet

## Application Security Groups (ASGs)

Logical grouping of NICs by application role. Use as source/destination in NSG rules instead of IP addresses.

### How ASGs Work
- Group NICs by role: `ASG-WebServers`, `ASG-AppServers`, `ASG-DbServers`
- Rule example: Source: ASG-WebServers → Destination: ASG-AppServers, Port: 8080, Allow
- No IP address management needed — ASGs scale automatically as VMs added/removed
- NICs can belong to multiple ASGs
- ASGs and the NSG referencing them must be in the same region

### ASG Gotchas
- Cannot mix ASGs with IP ranges in the same rule — use separate rules
- ASGs and NSG must be in the same VNet
- ASG membership is per-NIC, not per-VM (multi-NIC VMs can have different ASG memberships)

## NSG Design Patterns

### Subnet-Level NSGs (Standard Approach)
- Apply an NSG to every subnet — no exceptions
- Baseline: deny all inbound from Internet, explicitly allow required traffic
- Every subnet gets its own NSG (don't share NSGs across subnets for manageability)

### ASG-Based Multi-Tier Pattern
```
Web tier NSG:
  Allow Internet → ASG-Web on ports 80, 443
  Deny all other inbound

App tier NSG:
  Allow ASG-Web → ASG-App on port 8080
  Deny all other inbound

DB tier NSG:
  Allow ASG-App → ASG-Db on port 1433
  Deny all other inbound
```

### NIC-Level NSGs
- Additional granularity beyond subnet NSGs
- Rarely needed if subnet NSGs with ASGs are well-designed
- Use when specific VMs in a subnet need different rules

### Landing Zone Pattern
- Platform team enforces: deny SSH/RDP from Internet via Azure Policy
- Workload teams add specific allows within their NSGs
- NSG association enforced by policy — subnets cannot exist without NSGs

## NSG Flow Logs

- Enable for troubleshooting and compliance — logs all traffic through NSGs
- VNet flow logs preferred over NSG flow logs (simpler scope, less duplication)
- Send to Log Analytics workspace or Storage Account
- Traffic Analytics: visualization layer on top of flow logs — patterns, hotspots, threats
- Retention: configure based on compliance requirements

## Best Practices

- Every subnet gets an NSG — enforce via Azure Policy
- Start with deny-all inbound, explicitly allow required traffic
- Use ASGs over IP ranges for application-level rules
- Use service tags for Azure service traffic
- Document all NSG rules with description field
- Review rules quarterly — remove unused rules
- Enable flow logs and Traffic Analytics on all production VNets
- Leave priority gaps (100, 200, 300) for future insertions
- Name NSGs clearly: `nsg-<workload>-<tier>-<region>`
