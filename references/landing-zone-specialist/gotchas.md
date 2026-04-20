## Landing Zone Gotchas

Operational surprises and non-obvious behaviors in Azure landing zone management.

> Source: Microsoft Cloud Adoption Framework, Azure Resource Manager documentation

---

### Management Group Moves — Immediate Policy Inheritance

When you move a subscription to a new management group, it inherits the target management group's
policies and RBAC immediately. Existing resources subject to a `Deny` policy appear non-compliant
but are not retroactively rejected. However, any subsequent update to those resources will be
blocked by `Deny` policies. Resources under `DeployIfNotExists` appear non-compliant and require
manual remediation — they are not auto-remediated on move. User RBAC tokens may take up to 30
minutes to refresh after a management group move.

### Subscription Transfer — Tenant Change Breaks Everything

Transferring a subscription to a different Microsoft Entra tenant changes the identity provider.
All Azure RBAC assignments are permanently deleted. Managed identities (system-assigned and
user-assigned) become orphaned and non-functional. Key Vault access policies using Entra object
IDs break. Service principals and app registrations lose access. This is effectively a destructive
operation for identity — plan for full RBAC and identity reconfiguration after transfer.

### DINE Policies Require Managed Identity with Permissions

DeployIfNotExists and Modify policy effects create or alter resources on behalf of the policy
engine. This requires a managed identity with the necessary RBAC permissions at the target scope.
If the managed identity lacks permissions, the policy evaluation succeeds but remediation tasks
fail silently. Always verify the policy assignment's managed identity has the correct role
(typically Contributor or a scoped custom role) at the assignment scope.

### ALZ Accelerator — Deployment Creates Many Resources, Cleanup Is Difficult

The ALZ portal accelerator, Bicep, and Terraform modules deploy management groups, policies,
policy assignments, role assignments, Log Analytics workspaces, Automation accounts, and
networking resources. There is no "undo" or "tear down" button. Removing an ALZ deployment
requires manually deleting resources in reverse dependency order, removing policy assignments
before policy definitions, and handling management group deletion restrictions. Test in a
lab tenant first.

### Subscription Limits — 800 Resource Groups per Subscription

Each subscription supports a maximum of 800 resource groups. This limit is hard and cannot be
increased via support request. Workloads that programmatically create resource groups (per-tenant
SaaS, per-customer deployments) can hit this limit. Plan subscription scaling strategy for
high-density workloads and use subscription vending to provision additional subscriptions.

### Management Group Limits

Maximum 6 levels of depth (excluding root and subscription level). Maximum 10,000 management
groups per tenant. Management groups cannot be deleted if they contain subscriptions or child
management groups. Moving management groups requires write access on both the source parent and
the target parent. These limits rarely block enterprise designs but matter for ISVs and large
multi-tenant environments.

### Policy Assignment Scope — Lower Assignments Don't Override Higher Ones

Azure Policy is additive, not hierarchical-override. A policy assigned at a parent management
group and a different policy assigned at a child management group both apply. If the parent
assigns a Deny policy and the child assigns an Audit policy for the same condition, Deny wins
because both evaluate. You cannot "relax" a parent policy at a child scope — you can only add
exemptions. Design policy strategy top-down with this in mind.

### Subscription Names — Changeable, but SubscriptionId Is Permanent

Subscription display names can be changed at any time. However, the subscriptionId (GUID) is
permanent and immutable for the lifetime of the subscription. Scripts, IaC templates, and
automation that reference subscriptions should always use subscriptionId, never display name.
Naming conventions for subscriptions are cosmetic and should not be relied upon for logic.

### Resource Locks — Prevent Deletion but Not Cost Accrual

CanNotDelete locks prevent resource deletion but do not stop the resource from running and
accruing charges. A forgotten VM with a CanNotDelete lock still incurs compute costs daily.
ReadOnly locks prevent modifications but can break resources that need to write to themselves
(e.g., storage accounts writing logs). Locks also block some management operations like scaling.
Use locks judiciously and audit them regularly.

### Cross-Subscription Resource Moves — Many Resources Don't Support It

Many Azure resource types do not support cross-subscription move. Notable examples: resources
with managed identities referencing the source subscription, VMs with managed disks in certain
configurations, resources linked to a virtual network that can't be moved, and App Service
Environments. Always check the [Move operation support](https://learn.microsoft.com/azure/azure-resource-manager/management/move-support-resources)
matrix before planning cross-subscription migrations. Re-deployment is often the safer path.

---

**Key takeaway:** Test landing zone changes (policy assignments, subscription moves, accelerator
deployments) in a non-production management group hierarchy or lab tenant before applying to
production.
