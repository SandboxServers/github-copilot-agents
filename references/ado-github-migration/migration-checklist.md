# ADO → GitHub Migration Checklist

## Phase 1: Pre-Migration Assessment

### Pipeline & Build Inventory
- [ ] Inventory all YAML pipelines — count, complexity, template usage
- [ ] Inventory all classic build definitions (must convert to YAML first or directly to GHA)
- [ ] Inventory all classic release pipelines — stages, gates, approvals
- [ ] Document all template dependencies and shared template repositories
- [ ] Map all pipeline triggers (CI, PR, scheduled, pipeline resource triggers)
- [ ] Run `gh actions-importer audit` for automated pipeline assessment
- [ ] Run `gh actions-importer forecast` for runner usage estimation

### Service Connections & Authentication
- [ ] Inventory all service connections by type (Azure RM, Docker, Kubernetes, npm, NuGet, etc.)
- [ ] Identify which service connections use service principals vs managed identity
- [ ] Document all Azure subscription/resource group targets per connection
- [ ] Plan OIDC federated credential setup for each Azure target
- [ ] Identify non-Azure service connections (AWS, GCP, third-party) and plan GHA equivalents

### Variables & Secrets
- [ ] Inventory all variable groups — plain-text and Key Vault-linked
- [ ] Inventory all pipeline-level variables (per-pipeline and per-stage)
- [ ] Identify which variables are secrets vs. plain configuration
- [ ] Document Key Vault references and access policies
- [ ] Plan GitHub secret/variable scope mapping (org, repo, environment)

### Environments & Approvals
- [ ] Inventory all ADO environments with approval gates
- [ ] Document approval chains (who approves, how many, timeout)
- [ ] Identify environments with exclusive locks, checks, or resource-level checks
- [ ] Map ADO environments to planned GitHub environments
- [ ] Document business hours, branch filters, and other gate conditions

### Infrastructure & Agents
- [ ] Inventory all agent pools (Microsoft-hosted images, self-hosted pools)
- [ ] Document self-hosted agent OS, installed tools, network access
- [ ] Identify any pool-specific demands or capabilities used in pipelines
- [ ] Plan GitHub runner strategy: hosted, larger runners, self-hosted, runner groups
- [ ] Evaluate self-hosted runner infrastructure (ARC, VM scale sets, container-based)

### Artifacts & Feeds
- [ ] Inventory all Azure Artifacts feeds (NuGet, npm, Maven, Python, Universal)
- [ ] Document upstream sources and feed permissions
- [ ] Plan feed migration or continued use of Azure Artifacts from GHA
- [ ] Inventory pipeline artifacts — what's published, what's consumed downstream

### Code & Repo Assessment
- [ ] Inventory all repositories — size, branch count, contributor count
- [ ] Identify any TFVC repos requiring Git conversion
- [ ] Document branch policies per repo
- [ ] Identify repos with Git LFS needs
- [ ] Review CODEOWNERS / reviewer requirements

### Parity & Feature Gaps
- [ ] Identify parity gaps specific to this migration (see parity-gap-workarounds.md)
- [ ] Determine GitHub plan features needed (Team, Enterprise, GHAS)
- [ ] Evaluate GitHub Enterprise Managed Users vs GitHub.com identity
- [ ] Document any compliance/regulatory requirements affecting migration

---

## Phase 2: Migration Execution

### Repository Setup
- [ ] Create GitHub organization (if not existing)
- [ ] Configure organization-level settings (member privileges, default permissions)
- [ ] Create repository structure matching migration plan
- [ ] Migrate Git repos using `git clone --mirror` + `git push --mirror`
- [ ] Convert any TFVC repos to Git using git-tfs before migration
- [ ] Verify all branches, tags, and commit history migrated correctly

### Authentication & Secrets
- [ ] Create Entra ID app registrations for OIDC federated credentials
- [ ] Configure federated credentials for each environment (dev, staging, production)
- [ ] Create GitHub organization-level secrets (shared credentials)
- [ ] Create GitHub repository-level secrets (repo-specific credentials)
- [ ] Create GitHub environment-level secrets (environment-specific overrides)
- [ ] Create GitHub variables (non-secret configuration values)
- [ ] Test OIDC authentication with a simple workflow per environment

### Environments & Protection
- [ ] Create GitHub environments matching ADO environment plan
- [ ] Configure environment protection rules (required reviewers)
- [ ] Configure deployment branch/tag restrictions
- [ ] Set up wait timers where applicable
- [ ] Configure custom deployment protection rules (GitHub Apps) if using gates

### Workflow Conversion
- [ ] Create shared reusable workflow repository (`.github` or dedicated repo)
- [ ] Convert pipelines in order: simple CI → complex CI → CD → scheduled → triggered
- [ ] Use `gh actions-importer dry-run` for initial conversion of each pipeline
- [ ] Manually refine converted workflows — importer output is a starting point only
- [ ] Replace ADO tasks with equivalent GHA actions (see mapping references)
- [ ] Convert template references to reusable workflow / composite action calls
- [ ] Test each workflow in isolation before enabling triggers

### Code Quality & Security
- [ ] Enable GitHub Advanced Security (GHAS) on all repos
- [ ] Configure CodeQL analysis workflows
- [ ] Enable secret scanning and push protection
- [ ] Configure Dependabot alerts and security updates
- [ ] Set up Dependabot version updates for action pinning (.github/dependabot.yml)
- [ ] Migrate SonarQube/SonarCloud configuration to GHA workflows

### Branch Protection & Policies
- [ ] Configure branch rulesets (preferred) or branch protection rules
- [ ] Set up required status checks matching converted workflow jobs
- [ ] Create CODEOWNERS files based on ADO reviewer policies
- [ ] Configure merge method restrictions (squash, merge commit, rebase)
- [ ] Enable conversation resolution requirement where applicable

---

## Phase 3: Post-Migration Validation

### Functional Validation
- [ ] All workflows trigger correctly on push events
- [ ] All workflows trigger correctly on pull_request events
- [ ] Scheduled workflows run at expected times (note: cron is UTC in GHA)
- [ ] Manual workflow_dispatch triggers work with expected inputs
- [ ] Repository_dispatch and cross-workflow triggers function correctly

### Build & Test Validation
- [ ] All builds compile/succeed with same results as ADO
- [ ] All tests pass and results are visible (check annotations or PR comments)
- [ ] Code coverage is published and tracked
- [ ] Code analysis (CodeQL, SonarCloud) runs and reports findings

### Deployment Validation
- [ ] All deployments reach target environments successfully
- [ ] OIDC authentication works for all Azure targets
- [ ] Approval gates trigger and function correctly
- [ ] Environment-specific secrets/variables resolve properly
- [ ] Deployment logs show expected output and no auth errors

### Integration Validation
- [ ] Artifacts are produced and accessible
- [ ] Package feeds (NuGet, npm) are published/consumed correctly
- [ ] Notifications and integrations work (Slack, Teams, email)
- [ ] Status badges are updated and linked in README
- [ ] PR status checks block merges on failure as expected

### Parallel Run Comparison
- [ ] Run ADO and GHA in parallel for 1-2 sprints
- [ ] Compare build outputs, test results, deployment outcomes
- [ ] Verify equivalent coverage, analysis findings, artifact contents
- [ ] Document any behavioral differences for team awareness

---

## Phase 4: Cutover

### Switch Execution
- [ ] Disable ADO pipeline triggers (do NOT delete pipelines yet)
- [ ] Enable GHA workflow triggers as primary
- [ ] Update any webhook integrations to point to GitHub
- [ ] Update deployment notifications to reference GHA
- [ ] Update status badges in documentation

### Burn-In Period (1-2 Sprints)
- [ ] Monitor all workflows daily for failures or regressions
- [ ] Track runner usage and optimize concurrency settings
- [ ] Collect team feedback on GHA developer experience
- [ ] Address any edge cases discovered during burn-in
- [ ] Keep ADO pipelines intact as rollback option

### Decommission
- [ ] Archive ADO pipelines (disable, rename with [MIGRATED] prefix)
- [ ] Remove or rotate ADO service connection credentials
- [ ] Update team onboarding documentation to reference GHA
- [ ] Conduct team training session on GitHub Actions workflows
- [ ] Archive ADO project or set read-only (if fully migrated)

---

## Rollback Plan Template

If critical issues are discovered post-cutover:

1. **Re-enable ADO pipeline triggers** (they were disabled, not deleted)
2. **Disable GHA workflow triggers** (remove push/PR triggers or disable workflows)
3. **Revert webhook integrations** to point back to ADO
4. **Communicate rollback** to team with timeline for re-attempt
5. **Document failures** — what broke, root cause, fix plan
6. **Re-execute migration** with fixes after root cause is resolved

**Key principle:** Never delete ADO pipelines until the GHA equivalents have been validated
through a full confidence period (minimum 2 sprints of production use).
