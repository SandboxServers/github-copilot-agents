# GitHub Actions Anti-Patterns — Deep Reference

## Monolithic Workflow Files

One massive workflow file doing build, test, lint, deploy, and notify. Becomes unmaintainable and impossible to reuse across repositories.

**Fix:** Break into reusable workflows with `workflow_call`. Each workflow has a single responsibility.

```yaml
# BAD: 500-line ci-cd.yml doing everything
# GOOD: Separate reusable workflows
jobs:
  build:
    uses: ./.github/workflows/build.yml
  deploy:
    uses: ./.github/workflows/deploy.yml
    needs: build
```

## Hardcoded Secrets in Workflow Files

Embedding credentials, tokens, or connection strings directly in YAML files. They end up in version control history permanently.

**Fix:** Use GitHub Secrets at repository or environment level. Reference via `${{ secrets.SECRET_NAME }}`. For cloud auth, prefer OIDC over secrets entirely.

## No Concurrency Control

Multiple deployments racing to the same environment when commits are pushed rapidly. Can leave infrastructure in an inconsistent state.

**Fix:** Use concurrency groups to serialize deployments.

```yaml
concurrency:
  group: deploy-${{ github.ref }}
  cancel-in-progress: false  # Don't cancel active deployments
```

## Using Outdated Action Versions

Referencing `actions/checkout@v2` or older versions. Older versions miss security patches and may use deprecated Node.js runtimes (Node 12/16). GitHub logs deprecation warnings that become errors.

**Fix:** Keep actions updated. Pin to full SHA of the latest major version.

```yaml
# BAD
- uses: actions/checkout@v2

# GOOD — pinned to SHA of v4
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```

## Self-Hosted Runners Without Security Hardening

Running self-hosted runners on shared infrastructure without isolation. Malicious workflows from forks can execute arbitrary code on your infrastructure.

**Fix:** Use ephemeral runners (`--ephemeral` flag), run in containers, never use self-hosted runners on public repos, implement network segmentation.

## No Artifact Retention Policy

Default artifact retention is 90 days. Without explicit policies, storage costs grow unbounded, especially with large build outputs.

**Fix:** Set retention explicitly. Use repository/org-level defaults.

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/
    retention-days: 7  # Explicit short retention
```

## Too Many workflow_dispatch Inputs

Workflows with 10+ `workflow_dispatch` inputs become unwieldy. The GitHub UI input form is limited and error-prone for complex configurations.

**Fix:** Accept a single config file path or use repository variables. Read configuration from a YAML/JSON file in the repo.

## Not Using OIDC for Cloud Authentication

Still storing long-lived cloud credentials (AWS keys, Azure service principal secrets) as GitHub Secrets. These credentials can leak, expire, and require rotation.

**Fix:** Use OIDC federated credentials for Azure, AWS, and GCP. No secrets to rotate.

```yaml
permissions:
  id-token: write
steps:
  - uses: azure/login@v2
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

## Running on Push to Main Without Branch Protection

Triggering deployments on every push to `main` without requiring PR reviews or status checks. A single broken commit deploys directly to production.

**Fix:** Enable branch protection rules. Require PR reviews, passing status checks, and linear history. Use environment protection rules for production deployments.

## No Caching for Package Managers

Every workflow run downloads all dependencies from scratch. Wastes minutes of billable time per run.

**Fix:** Use built-in caching in setup actions or explicit cache action.

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: 'npm'  # Built-in caching
```

## Shell Scripts Without `set -e`

Inline `run:` steps using bash without `set -e`. Commands can fail silently and the step still reports success.

**Fix:** Set shell defaults or use `set -euo pipefail` in every script block.

```yaml
defaults:
  run:
    shell: bash
steps:
  - run: |
      set -euo pipefail
      npm ci
      npm test
```

## Not Pinning Actions to SHA

Using `@v4` or `@main` tags which are mutable. A compromised action can push a malicious update to the same tag. This is a real supply chain attack vector (e.g., the `tj-actions/changed-files` incident).

**Fix:** Always pin to the full commit SHA. Use tools like Dependabot or Renovate to automate SHA updates.

```yaml
# BAD — mutable tag
- uses: actions/checkout@v4

# GOOD — immutable SHA
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```
