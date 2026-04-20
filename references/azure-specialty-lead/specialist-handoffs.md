## Specialist Handoffs

When one specialist's output becomes another specialist's input, the handoff must be explicit. Implicit handoffs — "they'll figure it out" — create gaps that surface in production.

### What a Clean Handoff Documents

Every handoff between specialists must capture:
- **What was decided:** The specific design choices made (e.g., "three subnets: compute, data, management")
- **What depends on it:** Which downstream specialists need this information and why
- **What's next:** The specific action the receiving specialist should take
- **Interface contract:** The concrete values the next specialist needs

### Interface Contracts

Handoffs are only useful when they include the actual values, not just intentions:

| Contract Element | Example |
|-----------------|---------|
| IP ranges / CIDR blocks | `10.1.0.0/16` for hub VNet, `10.2.0.0/16` for spoke |
| DNS names | `*.privatelink.database.windows.net` resolves via private DNS zone |
| Endpoints | `https://kv-shared-prod-eastus2.vault.azure.net` |
| Credentials / identity | Managed identity `id-app-orders-prod` with Key Vault Secrets User role |
| Resource IDs | `/subscriptions/.../resourceGroups/rg-platform-prod/providers/...` |
| Configuration values | Connection string format, timeout values, retry policies |

### Handoff Template

```
## Handoff: [Source Specialist] → [Receiving Specialist]
**Date:** [date]
**Context:** [what problem this handoff relates to]

### Decisions Made
- [decision 1]
- [decision 2]

### Interface Contract
| Item | Value |
|------|-------|
| [contract item] | [concrete value] |

### Dependencies
- [what the receiving specialist's work depends on from this handoff]

### Next Steps
- [specific action for receiving specialist]
```

### Common Handoff Patterns

**Network → Compute:**
Network engineer provides VNet topology, subnet CIDRs, NSG rules, and private DNS zone configuration. Compute engineer uses these to configure VNet integration, deploy into correct subnets, and validate DNS resolution for private endpoints.

**Compute → Database:**
Compute engineer provides managed identity details and expected connection patterns (connection pooling, retry logic, expected throughput). Database specialist uses these to configure firewall rules, RBAC assignments, and performance tier.

**Database → Monitoring:**
Database specialist provides key metrics to observe (DTU consumption, connection count, replication lag, deadlocks). Monitoring engineer configures diagnostic settings, alert rules, and dashboard panels for these metrics.

**Integration → Compute:**
Integration architect provides message schemas, queue/topic names, and expected throughput patterns. Compute engineer sizes consumers accordingly and configures autoscaling triggers based on queue depth.

**Network → Everything:**
Every specialist receives the network handoff. Private endpoint DNS names, allowed outbound destinations, NSG constraints, and VNet integration requirements. This is the most critical handoff — get it wrong and every downstream specialist troubleshoots network connectivity issues.

### The Specialty Lead's Role

The Specialty Lead reviews every handoff for completeness. Missing contract elements get caught before they become "works on my machine" problems. When a handoff is unclear, the Specialty Lead convenes both specialists to resolve ambiguity immediately — not after deployment.
