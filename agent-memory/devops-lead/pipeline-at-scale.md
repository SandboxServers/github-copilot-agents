## Pipelines at Scale — Managing Hundreds of Pipelines

When an organization grows beyond a handful of repositories, pipeline management becomes an operational challenge. Managing 200+ pipelines requires deliberate strategy around templates, agents, monitoring, and cost.

### Template Versioning Strategy

Shared templates are the foundation of scale, but they introduce a versioning problem:
- **Semantic versioning**: Use major.minor.patch versioning for template repos. Major bumps signal breaking changes, minor adds features, patch fixes bugs
- **Version pinning**: Every consuming pipeline references a specific template version. No pipeline should reference `main` or `latest` in production
- **Upgrade cadence**: Establish a regular upgrade window (e.g., quarterly) where teams evaluate and adopt new template versions
- **Automated upgrade PRs**: Tooling that opens PRs in consuming repos when new template versions are available, similar to Dependabot for dependencies

### Breaking Changes Communication

Template changes that break consumers require careful handling:
- Announce breaking changes in advance through internal communication channels (Teams, Slack, email)
- Publish migration guides alongside new major versions — show the before and after
- Maintain the previous major version with critical fixes for a defined support period (e.g., 90 days)
- Use deprecation warnings in templates — emit pipeline warnings when consumers use deprecated features
- Track adoption of new versions and follow up with teams that fall behind

### Shared Variable Groups and Configuration

At scale, configuration management becomes critical:
- **Organizational variables**: Shared across many pipelines — registry URLs, tenant IDs, environment names
- **Team variables**: Scoped to a team's pipelines — service connection names, resource group prefixes
- **Pipeline variables**: Specific to a single pipeline — application-specific settings
- Avoid secrets in variable groups where possible — use Key Vault references or OIDC to eliminate stored credentials
- Document variable group ownership and review access regularly

### Agent Pool Management

**Microsoft-hosted agents** are the default choice:
- Zero maintenance overhead — Microsoft manages patching, updates, and capacity
- Pay per minute of build time with parallel job limits
- Clean environment every run eliminates state contamination
- Limited customization — cannot install specialized tools or access private networks directly

**Self-hosted agents** are necessary when:
- Builds require access to private networks (on-premises databases, internal registries)
- Specialized hardware or software is needed (GPU builds, licensed tools)
- Build performance requires persistent caches or warm environments
- Regulatory requirements mandate builds run on controlled infrastructure

For self-hosted agents at scale:
- Use VM scale sets or container-based agents that auto-scale with demand
- Implement agent image pipelines — build and test agent images through CI/CD like any other artifact
- Monitor agent health, availability, and utilization — dead agents in the pool slow everyone down
- Rotate agent images regularly to pick up OS patches and tool updates

### Pipeline Monitoring

Treat pipeline infrastructure like production infrastructure:
- **Duration tracking**: Monitor build and deployment times per pipeline. Set alerts for significant increases that indicate degradation
- **Failure rate**: Track pipeline success/failure rates. A consistently failing pipeline wastes developer time and blocks delivery
- **Queue time**: Time between pipeline trigger and execution start. High queue times indicate insufficient agent capacity
- **Flaky test detection**: Identify tests that intermittently fail and pass — flaky tests erode trust in the pipeline
- **Resource utilization**: Agent CPU, memory, and disk usage during builds — identify pipelines that need more resources or optimization

Build dashboards that surface the worst-performing pipelines so platform teams can prioritize optimization efforts.

### Cost Management

Pipeline costs scale with organizational growth and can become significant:
- **Parallel jobs**: Each concurrent pipeline run requires a parallel job slot. Monitor utilization to right-size purchases — too few causes queuing, too many wastes budget
- **Build minutes**: Microsoft-hosted agents charge per minute. Optimize pipeline duration to reduce cost — cache dependencies, parallelize stages, skip unnecessary steps
- **Storage**: Artifact retention policies prevent unbounded storage growth. Set retention to the minimum needed for compliance and rollback
- **Self-hosted infrastructure**: Agent VMs, scale sets, and networking have their own cost. Track cost per build minute for self-hosted versus Microsoft-hosted to make informed decisions
- **Idle capacity**: Self-hosted agents running 24/7 but only used during business hours waste money. Use scale-to-zero configurations where supported

### Governance at Scale

- **Pipeline creation standards**: New repos should use shared templates — provide a repo scaffold that includes a pre-configured pipeline
- **Compliance scanning**: Automated checks that all pipelines meet organizational standards (approved agent pools, required security stages, proper environment gates)
- **Orphaned pipeline cleanup**: Regularly audit for pipelines attached to archived or abandoned repos
- **Access reviews**: Pipeline service connections and secrets should be reviewed quarterly — remove access that is no longer needed