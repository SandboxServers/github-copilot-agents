## Greenfield vs Brownfield

### Greenfield (New Tenant)

```
Step 1: Tenant setup → Entra ID, billing, initial admin accounts
Step 2: Management group hierarchy → Deploy ALZ reference structure
Step 3: Platform landing zone → Connectivity, Identity, Management subscriptions
Step 4: Network foundation → Hub (spoke or VWAN), DNS, ExpressRoute/VPN
Step 5: Governance baseline → Azure Policy (audit mode), tagging strategy
Step 6: Identity foundation → PIM, conditional access, break-glass accounts
Step 7: Monitoring baseline → Log Analytics, Defender for Cloud, Sentinel
Step 8: Subscription vending → Automation for app team self-service
Step 9: First workload → App team deploys into vended subscription
Step 10: Iterate → Promote policies to enforce, add policies, refine
```

### Brownfield (Existing Mess)

```
Step 1: Assess current state → How many subscriptions? MG hierarchy? Policies?
Step 2: Inventory → What's deployed, who owns it, what's the dependency map?
Step 3: Design target state → ALZ reference, adapted to existing constraints
Step 4: Deploy platform alongside existing → Don't rip and replace
Step 5: Move subscriptions into new MG hierarchy → Start with non-prod
Step 6: Apply policies in audit mode → Find non-compliance without breaking
Step 7: Remediate → Work with teams to fix issues
Step 8: Migrate connectivity → Move to hub-spoke/VWAN (most disruptive step)
Step 9: Enforce policies → Graduate audit → deny
Step 10: Subscription vending → New subscriptions go through proper process
```

### Brownfield Red Flags

- Subscriptions under Tenant Root Group (no MG hierarchy)
- Service principals with Owner on subscription (credential-based)
- No Azure Policy at all
- Multiple disconnected VNets with overlapping address spaces
- Public IPs on everything
- No diagnostic settings → no visibility
- Shared subscriptions with multiple unrelated workloads
- Classic resources (ASM) still running
