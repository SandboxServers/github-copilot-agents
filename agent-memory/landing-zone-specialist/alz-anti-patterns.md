## ALZ Anti-Patterns

### Flat Management Group Structure

Using a single level of management groups (or no management groups at all) eliminates the ability
to apply different policy sets to different workload tiers. Corp workloads need stricter policies
than sandbox environments. Without hierarchy, you either over-restrict sandboxes or under-protect
production. Follow the ALZ reference hierarchy: Platform, Landing Zones (Corp/Online), Sandbox,
Decommissioned.

### No Subscription Vending

Manual subscription provisioning (email request, ticket queue, weeks of waiting) leads to shared
subscriptions, inconsistent configuration, and teams bypassing the process entirely. Implement
automated subscription vending so teams get properly configured subscriptions in minutes, not
weeks. Every subscription should arrive with correct MG placement, RBAC, networking, and policies.

### Policies in Deny Mode from Day One

Deploying Deny policies immediately blocks existing deployments on their next update cycle and
prevents teams from deploying at all until they understand the new rules. This creates backlash
and often results in policies being removed entirely. Always start with Audit mode, communicate
findings, allow remediation time, then promote to Deny after teams are prepared.

### Single Subscription for Everything

Putting all workloads in one subscription eliminates blast radius isolation, makes RBAC
unmanageable (everyone needs access to everything), prevents meaningful cost allocation, and hits
subscription-level resource limits. Use one subscription per workload per environment as the
default pattern. Subscriptions are free — use them as isolation boundaries.

### No Network Segmentation

Deploying all workloads into a flat network (single VNet, no NSGs, no firewall) removes the
ability to control east-west traffic, detect lateral movement, or isolate compromised workloads.
Use hub-spoke topology with NSGs on every subnet, Azure Firewall for north-south and east-west
inspection, and network policies that deny traffic not explicitly allowed.

### Skipping the Identity Subscription

Deploying domain controllers (or Entra Connect servers) directly into application subscriptions
mixes platform infrastructure with workload resources. Application teams should not manage
identity infrastructure. Place identity services in a dedicated Identity subscription under the
Platform management group, managed by the central identity team.

### No Sandbox Environment

Without a sandbox, developers experiment in production landing zones, creating non-compliant
resources, exposing test data, and consuming production budgets. Provide sandbox subscriptions
with relaxed policies, budget caps, and no connectivity to production networks. This channels
experimentation into a safe environment.

### Over-Customizing ALZ

Heavily modifying the ALZ reference architecture (custom management group hierarchies that don't
map to the reference, removing default policies, restructuring platform subscriptions) creates
drift from the well-documented and community-tested pattern. When Microsoft updates the ALZ
reference or accelerators, custom implementations cannot easily adopt improvements. Customize
within the framework (add child MGs, add policies) rather than replacing the framework itself.