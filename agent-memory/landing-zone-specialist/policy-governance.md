## Policy-Driven Governance

Azure Policy is the enforcement mechanism for landing zone guardrails. Policies are assigned at the
management group level and inherited by all child subscriptions, ensuring consistent governance
without requiring each team to configure compliance individually.

### Policy Effects

| Effect | Behavior | When to Use |
|--------|----------|-------------|
| **Audit** | Logs non-compliance, allows deployment | Initial rollout, visibility gathering |
| **Deny** | Blocks non-compliant deployments | After audit phase confirms readiness |
| **DeployIfNotExists (DINE)** | Auto-creates missing related resources | Diagnostic settings, monitoring agents |
| **Modify** | Alters resource properties during deployment | Tag inheritance, TLS version enforcement |
| **Append** | Adds properties to a resource | Adding required tags if missing |
| **AuditIfNotExists** | Audits when a related resource is missing | Check for NSG on subnet without blocking |

### Default ALZ Policy Categories

**Tagging policies** enforce cost allocation and ownership. Require tags like CostCenter, Owner,
Environment, and Application on resource groups. Use tag inheritance (Modify effect) to propagate
resource group tags to child resources automatically.

**Encryption policies** ensure data protection at rest and in transit. Enforce HTTPS-only on storage
accounts, require minimum TLS 1.2, enforce disk encryption on VMs, require encryption on SQL
databases, and deny unencrypted PaaS services.

**Private endpoint policies** restrict PaaS services from exposing public endpoints. Deny public
network access on Storage, SQL, Key Vault, Cosmos DB, and other services. Audit or deny resources
that lack private endpoint connections.

**Unused resource audit** identifies waste. Audit unattached managed disks, unused public IPs,
empty resource groups, and stopped VMs running longer than a defined period.

**DINE for diagnostics** automatically configures monitoring. Deploy diagnostic settings to send
platform logs to the central Log Analytics workspace. Deploy Microsoft Defender for Cloud on all
subscriptions. Deploy Azure Monitor Agent on VMs.

### Assignment Strategy

Assign policies at the **management group level**, not at individual subscriptions. This ensures:
- New subscriptions automatically inherit governance
- Consistent enforcement across all workloads in the same tier
- Platform MG gets platform-specific policies (hub networking rules)
- Landing Zones MG gets workload guardrails (no public IPs, required tags)
- Corp MG gets stricter policies (deny public endpoints)
- Online MG gets policies allowing controlled public access

### Exemption Process

Exemptions are necessary but must be governed:
- Require a documented business justification for every exemption
- Set expiration dates on all exemptions (30, 60, or 90 days)
- Assign an owner responsible for remediation before expiration
- Review exemptions monthly in governance review meetings
- Track exemptions in a central register (Azure Policy compliance dashboard)
- Expired exemptions auto-revert to enforcement

### Remediation Tasks

For existing resources that predate policy assignment:
1. Deploy policies in **Audit** mode first to assess blast radius
2. Review compliance dashboard to identify non-compliant resource counts
3. Create **remediation tasks** for DINE policies (deploys missing resources)
4. Work with application teams to fix Deny-eligible violations manually
5. Batch remediation by resource type (all storage accounts, then all VMs)
6. Validate compliance percentage reaches target before switching to Deny
7. Switch to Deny only after teams have had time to remediate

### Phased Rollout

```
Week 1-4:  Deploy all policies in Audit/AuditIfNotExists mode
Week 4-8:  Run remediation tasks, grant exemptions with expiration dates
Week 8-12: Promote high-confidence policies to Deny/DINE
Week 12+:  Add new policies in Audit, repeat the promotion cycle
```

Anti-pattern: deploying Deny policies on day one breaks existing deployments and creates backlash
that leads to policies being removed entirely. Always start with Audit.
