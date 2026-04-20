## Pipeline Architecture — Treating Pipelines as Products

Pipelines are products with users (development teams), requirements (reliability, speed, security), and a lifecycle (versioning, deprecation, support). A well-architected pipeline strategy scales across an organization without becoming a bottleneck.

### Template Architecture

Shared pipeline templates are the backbone of scalable CI/CD:
- **Centralized template repos**: Organization-wide repositories containing reusable pipeline definitions that teams consume
- **Versioned templates**: Every template change is tagged with a version. Consumers pin to a specific version and upgrade deliberately
- **Tested templates**: Templates have their own CI — validation pipelines that exercise templates with test inputs to catch regressions before consumers are affected
- **Parameterized design**: Templates accept inputs for customization rather than hard-coding assumptions. Sensible defaults reduce the configuration burden for common cases

In Azure DevOps, this means `extends` templates in dedicated repos referenced by pipeline YAML. In GitHub Actions, this means reusable workflows and composite actions published with version tags.

### Multi-Stage Pipeline Design

A production pipeline progresses through stages with increasing confidence:

**Build Stage**: Compile code, run unit tests, perform static analysis, generate artifacts. This stage must be fast — under 10 minutes for most applications. Every commit triggers this stage.

**Test Stage**: Run integration tests, API contract tests, and end-to-end tests against deployed artifacts. Use ephemeral environments where possible to avoid contention. Parallelize test suites to keep duration reasonable.

**Staging Stage**: Deploy to a production-like environment. Run smoke tests that validate the deployment succeeded. Perform security scans against the deployed application. Allow manual exploratory testing for significant changes.

**Production Stage**: Deploy using a progressive strategy — canary, blue-green, or rolling update depending on the application architecture. Monitor error rates and latency during rollout. Automated rollback triggers if health checks fail.

### Environment Promotion with Gates

Gates enforce quality and compliance between stages:
- **Automated gates**: Test pass rates, code coverage thresholds, security scan results, policy compliance checks
- **Manual gates**: Change advisory board approval for regulated environments, stakeholder sign-off for major releases
- **Time gates**: Soak time in staging before production promotion — let the deployment run under load before advancing
- **Business gates**: Feature flag readiness, documentation updates, support team notification

Gates should be as automated as possible. Manual gates introduce delay and should be reserved for situations where human judgment genuinely adds value.

### Artifact Management

Build once, deploy many:
- Build artifacts once in the CI stage and promote the same artifact through environments — never rebuild for each environment
- Store artifacts in a package registry (Azure Artifacts, GitHub Packages, or container registries) with retention policies
- Sign artifacts to ensure integrity — the artifact deployed to production is provably the same one that passed testing
- Include build metadata (commit SHA, build number, pipeline URL) in artifact tags for traceability

### Pipeline Security

Pipelines have access to production credentials, deployment targets, and sensitive configuration. They must be secured accordingly:
- **Least privilege**: Pipeline service connections and identities have only the permissions required for their specific tasks
- **OIDC federation**: Use workload identity federation instead of stored secrets for cloud provider authentication — no long-lived credentials in pipeline configuration
- **No stored secrets**: Secrets come from key vaults or secret stores at runtime, never hard-coded in pipeline YAML or variable groups
- **Branch protection**: Production deployment pipelines only run from protected branches with required reviews
- **Audit trail**: All pipeline modifications, approvals, and deployments are logged and reviewable

### Branch Strategies and Pipeline Implications

Branch strategy directly shapes pipeline design:

**Trunk-based development**: All developers commit to main (or short-lived feature branches merged within a day). The pipeline runs on every commit to main with automatic progression to production. Requires strong CI and feature flags for incomplete work.

**GitFlow or release branches**: Feature branches merge to develop, release branches cut from develop, production deploys from release branches. Pipelines are more complex — different stages trigger on different branch patterns. Longer feedback loops.

**Environment branches**: Branches represent environments (dev, staging, production). Merging to a branch triggers deployment to the corresponding environment. Simple to understand but creates merge conflicts and makes rollback complicated.

The DevOps lead should advocate for trunk-based development where feasible — it aligns with high-performing delivery practices and simplifies pipeline architecture.

### Pipeline as Code Principles

- **Version controlled**: Pipeline definitions live in the repository alongside the application code they build and deploy
- **Reviewed**: Pipeline changes go through pull request review with the same rigor as application code changes
- **DRY**: Common patterns are extracted into templates rather than duplicated across repositories
- **Observable**: Pipeline runs emit metrics (duration, pass rate, queue time) that feed operational dashboards
- **Self-documenting**: Stage names, job names, and comments make the pipeline intent clear without external documentation