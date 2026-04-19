## Deployment Stacks

### Overview
Deployment stacks manage Azure resources as a single unit. When a resource is removed from the template, it can be detached or deleted based on `actionOnUnmanage`.

### Key Operations
| Operation | Resource Group | Subscription | Management Group |
|---|---|---|---|
| **Create** | `az stack group create` | `az stack sub create` | `az stack mg create` |
| **Update** | Same create command | Same create command | Same create command |
| **Delete** | `az stack group delete` | `az stack sub delete` | `az stack mg delete` |
| **List** | `az stack group list` | `az stack sub list` | `az stack mg list` |
| **Show** | `az stack group show` | `az stack sub show` | `az stack mg show` |

### Deny Settings
Protect managed resources from unauthorized changes:
| Mode | What It Blocks |
|---|---|
| `none` | No protection |
| `denyDelete` | Blocks delete operations on managed resources |
| `denyWriteAndDelete` | Blocks both write and delete operations |

**Additional options:**
- `--deny-settings-apply-to-child-scopes` — extends protection to child resources
- `--deny-settings-excluded-actions` — RBAC operations excluded from deny (up to 200)
- `--deny-settings-excluded-principals` — Entra ID principals excluded from deny (up to 5)

### ActionOnUnmanage
When resources are removed from the template:
- `detachAll` — detach resources and resource groups (keep them, stop managing)
- `deleteResources` — delete resources but detach resource groups
- `deleteAll` — delete both resources and resource groups

### Known Limitations
- 800 stacks per scope, 2000 deny assignments per scope
- What-if not yet supported for stacks
- Can't delete Key Vault secrets through stacks
- Deleting resource groups bypasses deny assignments (use locks as secondary protection)
- Management group stacks can only deploy to same MG or child subscriptions
