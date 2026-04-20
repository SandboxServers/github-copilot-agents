## Governance

### Environment Strategy

| Environment | Purpose | Access | DLP Policy |
|---|---|---|---|
| **Default** | Lock down — no production apps | All users (can't restrict) | Block all premium connectors |
| **Production** | Managed, strict controls | Approved makers/admins only | Strict — business connectors only |
| **Sandbox** | Testing and experimentation | Developers and testers | Mirror production DLP |
| **Developer** | Individual maker environments | One maker per environment | Standard DLP |

- **Lock the default environment first** — every user has access, #1 governance gap
- Restrict environment creation to admins (Admin center → Tenant settings)
- Use environment groups to apply policies across multiple environments at once
- Auto-cleanup policy on sandbox environments (flag inactive after 90 days)

### DLP (Data Loss Prevention) Policies
- Classify connectors into three groups:
  - **Business** — connectors that can communicate with each other
  - **Non-Business** — connectors that can communicate with each other (but not Business)
  - **Blocked** — connectors that cannot be used at all
- Apply at tenant level (baseline) and override at environment level for specific needs
- **Start restrictive, loosen with justification** — far easier than tightening later
- Key rule: production Dataverse should not be connectable to unvetted SaaS apps
- Audit regularly — Microsoft adds new connectors that may not fit your policy
- HTTP connector is premium but very powerful — consider blocking in citizen dev environments

### Managed Environments
- Enhanced governance features for designated environments
- **Sharing limits** — restrict how widely apps can be shared (e.g., max 20 users)
- **Solution checker enforcement** — require solution checker to pass before deployment
- **Weekly digest emails** — automated reports on environment activity and usage
- **Maker welcome content** — show policies and guidelines when makers first open Power Apps
- **Data policies** — additional DLP controls specific to the managed environment
- Require Managed Environments for production and test environments

### CoE (Center of Excellence) Starter Kit

**Core Module**
- Inventory of all apps, flows, makers, connectors, and environments in tenant
- Sync flows that catalog Power Platform resources into Dataverse tables
- Power BI dashboard for tenant-wide visibility

**Governance Module**
- Compliance processes and inactivity notifications
- Cleanup workflows for orphaned and unused apps/flows
- Developer compliance center for policy acknowledgment

**Nurture Module**
- Maker community engagement and training tracking
- App showcases and innovation challenges
- Welcome emails and onboarding automation for new makers

- Built on Dataverse, deployed as solutions — requires dedicated environment
- Not a replacement for CoE people and process — it's tooling that supports human governance

### Admin Center Capabilities
- Manage environments, DLP policies, capacity, and tenant settings
- View analytics: active users, app usage, flow runs, error rates
- Manage maker permissions and environment assignments
- Configure tenant isolation to control cross-tenant connector access

### Tenant Isolation
- Control whether connectors can communicate with other tenants
- Block inbound (other tenants connecting to your data) or outbound (your users connecting to external)
- Configure per-connector or blanket policy
- Critical for regulated industries — prevent data exfiltration via connectors
