# GitHub Actions Best Practices — Deep Reference

## OIDC for All Cloud Authentication

Eliminate long-lived credentials. Use OpenID Connect (OIDC) federated identity for Azure, AWS, and GCP. Configure trust between GitHub and your cloud provider using federated credentials (e.g., Microsoft Entra Workload Identity Federation).

```yaml
permissions:
  id-token: write
  contents: read
steps:
  - uses: azure/login@v2
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

**Key:** Subject claims should use `repository_owner_id` and `repository_id` instead of `repository` name to survive repo renames.

## Pin Actions to Full Commit SHA

Never use mutable tags (`@v4`, `@main`) in production workflows. Tags can be force-pushed. Pin to the full 40-character SHA and comment the version for readability.

```yaml
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
- uses: actions/setup-node@cdca7365b2dadb8aad0a33bc7601856ffabcc48e # v4.3.0
```

Use Dependabot or Renovate to automate SHA updates when new versions release.

## Reusable Workflows for Shared Patterns

Centralize CI/CD logic into reusable workflows (`workflow_call`). Store them in a dedicated `.github` repository or shared org repo. Pass inputs and secrets explicitly.

```yaml
# Calling workflow
jobs:
  ci:
    uses: my-org/.github/.github/workflows/ci.yml@main
    with:
      node-version: '20'
    secrets: inherit  # Or pass explicitly
```

**Rule of thumb:** If two or more repos use the same workflow pattern, extract it into a reusable workflow.

## Concurrency Groups for Deployment Protection

Prevent concurrent deployments to the same environment. Use `cancel-in-progress: false` for deployments (never cancel a running deploy). Use `cancel-in-progress: true` for CI builds on the same branch.

```yaml
# Deployment — queue, don't cancel
concurrency:
  group: deploy-${{ inputs.environment }}
  cancel-in-progress: false

# CI — cancel superseded runs
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

## Environment Protection Rules

Use GitHub Environments for production deployments. Configure:
- **Required reviewers** — gate production deploys behind human approval
- **Wait timers** — enforce a delay before deployment proceeds
- **Branch restrictions** — only allow `main` to deploy to production
- **Environment secrets** — scope sensitive credentials to specific environments

## Caching for Package Managers

Use built-in caching in `setup-*` actions where available. Saves significant billable minutes.

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: 'npm'

- uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: 'pip'
```

For custom caching, use `actions/cache` with a well-designed key strategy including lockfile hashes.

## Security: Minimal GITHUB_TOKEN Permissions

Set workflow-level permissions to empty (`permissions: {}`), then grant per-job.

```yaml
permissions: {}  # Deny all at workflow level

jobs:
  build:
    permissions:
      contents: read
  deploy:
    permissions:
      contents: read
      id-token: write
```

**Never use third-party actions without reviewing their source.** Prefer actions from verified creators. Enforce org-level action allow lists.

## Artifact Management

- Set explicit `retention-days` on all uploads (default 90 days is excessive for most CI artifacts)
- Use artifact attestation for supply chain security (SLSA provenance)
- Separate test results from build artifacts — different retention needs
- Use `actions/attest-build-provenance` for generating attestations

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/
    retention-days: 7

- uses: actions/attest-build-provenance@v2
  with:
    subject-path: dist/
```

## Matrix Strategies for Multi-Platform Testing

Test across OS, runtime version, and configuration combinations. Use `fail-fast: false` to see all failures. Use `include`/`exclude` for special cases.

```yaml
strategy:
  fail-fast: false
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node: [18, 20, 22]
    exclude:
      - os: macos-latest
        node: 18  # Drop unsupported combo
```

## Branch Protection: Require Status Checks

Configure branch protection on `main` and release branches:
- **Require pull request reviews** before merge
- **Require status checks to pass** — link CI workflow jobs as required checks
- **Require linear history** — enforce squash or rebase merges
- **Restrict who can push** — limit direct push to emergency break-glass only
- **Require signed commits** where feasible

These rules ensure no code reaches production without passing CI and peer review.
