## Anti-Patterns in DevOps Practice

### Overview
DevOps transformations fail in predictable ways. These anti-patterns capture the
organizational, process, and technical failures that undermine delivery performance,
reliability, and team effectiveness. The existing `devops-anti-patterns.md` covers
CI/CD-specific failure modes — this file addresses the broader DevOps practice.

> **Source:** Patterns informed by Microsoft DevOps guidance
> ([What is DevOps?](https://learn.microsoft.com/devops/what-is-devops),
> [CI/CD pipelines](https://learn.microsoft.com/dotnet/architecture/cloud-native/devops),
> [End-to-end governance](https://learn.microsoft.com/devops/operate/governance-cicd)).

### Manual Deployments to Production
- SSH into servers, click through portals, run ad-hoc scripts
- Unrepeatable, unauditable, error-prone under pressure
- "It worked when I ran it" is not a deployment strategy
- **Fix:** Every production deployment flows through an automated pipeline with identical
  steps every time. If you cannot deploy with a merge or button press, the process is broken.

### No Environment Parity
- Dev runs on SQLite, staging on SQL Server, production on Cosmos DB
- Infrastructure provisioned by hand with undocumented drift between environments
- "Works in staging, breaks in production" is a regular occurrence
- **Fix:** Infrastructure as code with environment-specific parameters, never manual config.
  Use the same templates across all environments. Test against production-like infrastructure.

### Long-Lived Feature Branches
- Feature branches live for weeks or months, diverging from trunk
- Merge conflicts accumulate, integration risk deferred until the worst moment
- Teammates cannot build on each other's work
- **Fix:** Trunk-based development with short-lived branches (hours to days). Use feature
  flags for incomplete work. Integrate frequently, not painfully.

### No Automated Testing in Pipelines
- Pipelines build and deploy but never validate
- First real test happens in production via user complaints
- Regression bugs ship repeatedly because nothing catches them
- **Fix:** Shift left — unit tests, integration tests, and smoke tests run on every commit.
  A pipeline that only builds is a packaging script, not CI/CD.

### Shared Build Agents with Conflicting Tool Versions
- Multiple teams share agents with globally installed toolchains
- One team upgrades Node 18 → 20, breaking another team's build
- Agent state is mutable and unpredictable
- **Fix:** Immutable agent images or containerized build steps. Pin tool versions per
  pipeline. Treat agents like cattle, not pets.

### Pipeline as ClickOps
- Pipelines configured through portal UI clicks, not code
- No version history, no review process, no rollback capability
- Changes are invisible until something breaks
- **Fix:** Pipeline as code in the repository. YAML definitions reviewed in PRs, versioned
  alongside application code, and subject to the same governance.

### No Artifact Versioning or Traceability
- Builds produce artifacts with no version, no commit SHA, no provenance
- Cannot determine which code is running in production
- Rollback means "rebuild from whatever is in main right now"
- **Fix:** Tag every artifact with commit SHA, build ID, and pipeline run. Store artifacts
  in a versioned registry. Maintain a deployment manifest per environment.

### Deploying on Fridays Without a Rollback Plan
- Friday afternoon deployments with the team heading home
- No tested rollback mechanism — if it fails, improvise under pressure
- Incident response happens when nobody is available
- **Fix:** Every deployment has a tested rollback path — previous version redeploy,
  feature flag toggle, or traffic switch. Practice rollback regularly, not during incidents.

### No Monitoring of Pipeline Health and Duration Trends
- Nobody tracks whether pipelines are getting slower, flakier, or failing more often
- 45-minute builds become normal through gradual degradation
- Pipeline failures are noise, not signals
- **Fix:** Track pipeline duration, success rate, and queue time as first-class metrics.
  Alert on regressions. Treat pipeline health like production service health.

### Siloed Teams — Dev Throws Code Over the Wall to Ops
- Development writes code, tosses it to a separate operations team to deploy
- Ops has no context on what changed; dev has no visibility into production
- Blame flows in both directions, learning flows in neither
- **Fix:** Cross-functional teams own the full lifecycle. "You build it, you run it."
  Platform teams provide self-service tooling, not manual handoffs.

### Single Point of Failure — One Person Knows All Pipelines
- The "pipeline person" is the only one who understands the build system
- When they are on vacation, nobody can fix or modify pipelines
- Knowledge is tribal, not documented or shared
- **Fix:** Pipeline knowledge shared across the team through pairing, documentation,
  and rotation. At least two people can modify any pipeline at any time.

### Security Scanning Bolted On at the End
- Security review happens as a manual gate before production
- Findings arrive weeks after the code was written, context is lost
- Teams treat security as a blocker to route around, not a quality signal
- **Fix:** Shift left — dependency scanning, SAST, container scanning, and IaC policy
  checks run on every commit. Findings surface immediately to the developer who
  introduced them, not in a quarterly report.

> **See also:** [devops-anti-patterns.md](devops-anti-patterns.md) for CI/CD-specific
> failure modes including CI without CD, testing only in production, snowflake
> environments, and the DevOps Team anti-pattern.
