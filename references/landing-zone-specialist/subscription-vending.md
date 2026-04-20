## Subscription Vending

### Subscription as Unit of Management

In Azure Landing Zones, the subscription is the fundamental unit of management, scale, and policy
boundary. Each workload or environment gets its own subscription rather than sharing subscriptions
across unrelated workloads. This provides clear blast radius isolation, independent RBAC, separate
cost tracking, and policy inheritance from the correct management group.

### The Vending Machine Pattern

Subscription vending is a self-service (or near-self-service) process where application teams
request a subscription and receive it pre-configured and ready for workload deployment. This
eliminates manual provisioning bottlenecks and ensures every subscription meets organizational
standards from creation.

### Vending Inputs

When a team requests a subscription, they provide:
- **Workload name** — identifies the application or service
- **Environment** — dev, test, staging, or production
- **Owner** — team or individual responsible for the subscription
- **Budget** — monthly spend limit and alert thresholds
- **Connectivity requirements** — Corp (private only), Online (public-facing), or Sandbox (isolated)
- **Region** — primary deployment region
- **Compliance tier** — standard, regulated, or sovereign

### Vending Outputs

The automation delivers a subscription with:
- Placed in the **correct management group** (Corp, Online, or Sandbox)
- **Budget alerts** configured at 50%, 75%, 90%, and 100% of stated budget
- **Owner RBAC** assigned to the requesting team (Contributor on subscription)
- **VNet deployed and peered** to hub (for Corp/Online; isolated for Sandbox)
- **DNS configured** with link to central Private DNS zones
- **Diagnostic settings** sending activity logs to central Log Analytics
- **Policies inherited** automatically from management group placement
- **Tags applied** (CostCenter, Owner, Environment, Application)
- **Defender for Cloud** enabled via DINE policy inheritance

### IaC Implementation

Subscription vending is implemented as an IaC module invoked by a CI/CD pipeline:

**Terraform approach** — Use the `Azure/lz-vending` module from the Terraform Registry. Wrap it
in an organization-specific module that adds custom defaults (naming conventions, approved regions,
required tags). Trigger via pull request to a subscription request repository.

**Bicep approach** — Use the AVM `lz/sub-vending` pattern from `Azure/bicep-registry-modules`.
Deploy via Azure DevOps or GitHub Actions pipeline. Parameterize with a JSON/YAML request file
that maps to module inputs.

**Pipeline workflow:**
```
Request PR → Approval gate → Terraform/Bicep plan → Review → Apply → Notify team
```

### What Vending Replaces

Without vending, teams submit tickets, wait days or weeks, receive inconsistently configured
subscriptions, and often share subscriptions to avoid the wait. Vending reduces provisioning
from weeks to minutes while guaranteeing compliance from the start.
