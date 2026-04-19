# NSG and ASG Design

### Network Security Groups (NSGs)
- Stateful L3/L4 firewall — rules by source/destination IP, port, protocol
- Associate to subnet and/or NIC — dual association supported (subnet first, then NIC for inbound; NIC first, then subnet for outbound)
- Default rules: AllowVNetInBound (65000), AllowAzureLoadBalancerInBound (65001), DenyAllInBound (65500) — cannot delete, can override with lower priority
- Rules processed in priority order (100-4096) — lower number = higher priority, first match wins
- **Design best practice**: leave gaps in priority numbers (100, 200, 300) for future insertions
- Service tags: `VirtualNetwork`, `Internet`, `AzureLoadBalancer`, `AzureCloud`, `Storage`, `Sql`, `AzureMonitor`, etc.
- **Gotcha**: `VirtualNetwork` tag includes VNet address space, peered VNets, and on-premises addresses connected via gateways — not just the local VNet
- **Gotcha**: NSG rules mixing ASGs with IP ranges in the same rule is not supported — use separate rules
- **Gotcha**: ASGs and the NSG referencing them must be in the same VNet

### Application Security Groups (ASGs)
- Logical grouping of NICs — define rules by application role, not IP address
- Example: `AsgWeb`, `AsgLogic`, `AsgDb` — allow `AsgLogic → AsgDb:1433`, deny all other → `AsgDb:1433`
- NICs can belong to multiple ASGs (up to Azure limits)
- **Best practice**: use ASGs for multi-tier applications — micro-segmentation without explicit IPs
- **Best practice**: CAF recommends delegating subnet creation to landing zone owners, NSG association enforced by platform team via Azure Policy

### NSG Design Patterns
- **Per-subnet NSG**: baseline deny-all + explicit allows — standard approach
- **Per-tier NSG with ASGs**: web tier allows 80/443 from Internet, app tier allows from AsgWeb only, db tier allows from AsgApp only
- **Landing zone pattern**: platform team enforces deny SSH/RDP from Internet via policy, workload team adds specific allows
- **Flow logs**: enable VNet flow logs (preferred over NSG flow logs — simpler, less duplication) + traffic analytics for visibility
