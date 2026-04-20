## DevOps Best Practices

### Overview
These practices represent the operational patterns that high-performing delivery
organizations follow. They are not aspirational ideals — they are concrete,
implementable patterns with measurable outcomes. Each practice connects back to
DORA research on what actually drives delivery performance.

> **Source:** Informed by Microsoft DevOps guidance
> ([What is DevOps?](https://learn.microsoft.com/devops/what-is-devops),
> [shift-left testing](https://learn.microsoft.com/devops/develop/shift-left-make-testing-fast-reliable),
> [DevOps security](https://learn.microsoft.com/security/benchmark/azure/mcsb-v2-devop-security),
> [delivering quality services](https://learn.microsoft.com/devops/deliver/delivering-quality-services-with-devops)).

### DORA Metrics as North Star
- Deployment frequency: how often you ship to production
- Lead time for changes: commit to production elapsed time
- Change failure rate: percentage of deployments causing incidents
- Mean time to restore (MTTR): how fast you recover from failures
- **Practice:** Measure all four. Improve them as a system, not individually. A team
  that deploys frequently but breaks production constantly is not high-performing.

### Everything as Code
- Infrastructure defined in Terraform, Bicep, or ARM — never portal clicks
- Pipelines defined in YAML, versioned alongside application code
- Configuration managed through App Configuration, Key Vault references, or config-as-code
- Documentation lives in the repo, reviewed in PRs, deployed with the product
- **Practice:** If it is not in a repository, it does not exist. If it cannot be
  reviewed in a PR, it cannot be trusted. If it cannot be reproduced from code, it is fragile.

### Shift Left — Testing, Security, and Compliance Early in the Pipeline
- Unit tests run on every commit, not before release
- SAST, dependency scanning, and container scanning run in CI, not as a release gate
- IaC policy checks validate infrastructure before provisioning, not after
- Compliance evidence generated automatically, not assembled manually
- **Practice:** Move quality signals as early as possible. The cheapest bug to fix is
  the one caught in the developer's PR, not in production at 2 AM.

### Trunk-Based Development with Short-Lived Feature Branches
- Main branch is always deployable
- Feature branches live hours to days, not weeks to months
- Feature flags decouple deployment from release
- Code review happens on small, focused PRs — not thousand-line merges
- **Practice:** Integrate early and often. Small batches reduce risk, improve review
  quality, and keep the team moving in the same direction.

### Immutable Artifacts — Build Once, Deploy Everywhere
- A single build produces a versioned, immutable artifact
- The same artifact deploys to dev, staging, and production
- Environment differences handled through configuration injection, not rebuilds
- Artifact provenance tracked: commit SHA, build ID, pipeline run
- **Practice:** Never rebuild for a different environment. If the artifact changes
  between environments, you are not testing what you deploy.

### Progressive Delivery — Canary, Blue-Green, Feature Flags
- Canary deployments route a small percentage of traffic to the new version
- Blue-green deployments maintain two full environments for instant rollback
- Feature flags allow code deployment without feature activation
- Ring-based rollout exposes changes to internal users before external
- **Practice:** Decouple deployment (code ships) from release (users see it).
  Reduce blast radius and enable fast rollback without redeployment.

### Self-Service Pipelines — Templates and Shared Libraries
- Central platform team publishes pipeline templates for common patterns
- Application teams compose templates, not copy-paste YAML
- Template versioning allows teams to adopt updates at their own pace
- Guardrails (security scanning, compliance checks) baked into templates by default
- **Practice:** Make the right thing the easy thing. Teams should get security,
  testing, and deployment best practices for free by using shared templates.

### Pipeline Security — OIDC, No Persistent Secrets, Least Privilege
- Workload identity federation (OIDC) replaces stored service principal credentials
- Service connections scoped to specific environments and resource groups
- Pipeline permissions follow least privilege — no Contributor on the subscription
- Custom roles prevent destructive actions (delete resource groups, remove locks)
- **Practice:** Pipelines are automated service principals. Treat their access with
  the same rigor as production admin accounts. Audit regularly.

### Monitoring — Pipeline Health, Failure Rates, Deployment Frequency
- Pipeline duration, queue time, and success rate tracked as SLIs
- Deployment frequency and lead time measured automatically from pipeline data
- Alerts fire on pipeline duration regressions and failure rate spikes
- Dashboards visible to all teams, not hidden in admin portals
- **Practice:** Pipeline observability is production observability. If you cannot
  measure how fast and reliably you ship, you cannot improve it.

> **See also:** [dora-metrics.md](dora-metrics.md) for measurement deep-dive,
> [pipeline-architecture.md](pipeline-architecture.md) for template design,
> [devops-practice.md](devops-practice.md) for cultural foundations.
