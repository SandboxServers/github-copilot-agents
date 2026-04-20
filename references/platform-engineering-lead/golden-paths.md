## Golden Paths

### Definition
A golden path is a pre-built, opinionated, end-to-end workflow that handles a common
development scenario from start to finish. Golden paths make the right thing the easy thing
by encoding best practices, security controls, and operational requirements into a
streamlined developer experience.

### Principles
- **Optional but attractive** — teams can deviate, but the golden path should be clearly easier
- **Complete** — cover the full lifecycle, not just one step
- **Maintained** — the platform team owns and continuously updates paths
- **Feedback-driven** — improve paths based on what developers actually need
- **Escape hatches** — provide documented ways to customize when necessary

### Golden Path: New Microservice

| Step | What the Path Provides |
|---|---|
| Repository | Template repo with standard structure, linting, testing setup |
| CI/CD Pipeline | Pre-configured build, test, and deploy pipeline |
| Infrastructure | Terraform/Bicep module for compute, networking, identity |
| Monitoring | Application Insights, log analytics, standard dashboards |
| Security | Dependency scanning, secret detection, container scanning |
| Documentation | README template, API docs generation, ADR template |
| Networking | Service mesh integration, DNS registration, TLS certificates |

### Golden Path: New Azure Resource

| Step | What the Path Provides |
|---|---|
| Terraform Module | Approved module from internal registry with secure defaults |
| Policy Compliance | Pre-validated against Azure Policy and organizational standards |
| Networking | VNet integration, private endpoints, NSG rules |
| Identity | Managed identity, RBAC assignments, key vault integration |
| Monitoring | Diagnostic settings, metric alerts, log forwarding |
| Cost Controls | Approved SKUs, auto-shutdown for dev, budget alerts |

### Golden Path: New Team Onboarding

| Step | What the Path Provides |
|---|---|
| Subscriptions | Pre-provisioned Azure subscriptions via subscription vending |
| Access | Entra ID group creation, RBAC role assignments, PIM setup |
| Repositories | GitHub/ADO project with standard branch policies |
| CI/CD | Pipeline templates connected to environments |
| Tools | IDE configuration, CLI tools, local development setup |
| Training | Self-paced onboarding guide, office hours schedule |

### Keeping Paths Current
Golden paths are not set-and-forget. They require continuous investment:
- **Review quarterly** — are paths still aligned with organizational standards?
- **Track adoption** — which paths are used? Which are abandoned?
- **Gather feedback** — what's missing? What's frustrating?
- **Update dependencies** — keep base images, modules, and tools current
- **Deprecate gracefully** — when retiring a path, migrate existing users first

### When Teams Deviate
Teams will sometimes need to leave the golden path. This is expected and acceptable:
- Document the deviation and the reason
- The deviating team owns maintenance of their custom solution
- Compliance requirements still apply regardless of path
- Evaluate successful deviations for inclusion in future golden paths
- Extra maintenance burden naturally encourages return to supported paths
