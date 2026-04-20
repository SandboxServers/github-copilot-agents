# ADO → GitHub Actions: Parity Gap Workarounds

Every major feature gap between ADO Pipelines and GitHub Actions, with recommended workaround,
complexity estimate, and example YAML where applicable.

---

## 1. Deployment Strategies (Rolling / Canary)

**Gap:** ADO has built-in rolling deployment and canary strategy keywords. GHA has no native deployment strategy support.

**Workaround:** Implement via infrastructure-level strategies.
**Complexity:** Medium

- **Kubernetes:** Use native rolling update (default `kubectl apply`) or Helm `--set strategy.type=RollingUpdate`
- **Azure Container Apps:** Use revision splitting with `az containerapp revision` commands
- **Azure App Service:** Use deployment slots with swap (`azure/webapps-deploy` with `slot-name`)
- **Custom matrix:** Sequential deployments via matrix strategy with `max-parallel: 1`

```yaml
strategy:
  matrix:
    target: [region-1, region-2, region-3]
  max-parallel: 1          # Sequential deployment = rolling
steps:
  - uses: azure/webapps-deploy@v3
    with:
      app-name: 'myapp-${{ matrix.target }}'
      slot-name: staging
  - name: Health check
    run: curl -f https://myapp-${{ matrix.target }}-staging.azurewebsites.net/health
  - uses: azure/CLI@v2
    with:
      inlineScript: az webapp deployment slot swap -g rg -n myapp-${{ matrix.target }} -s staging
```

---

## 2. Pre/Post-Deployment Gates

**Gap:** ADO has built-in pre/post-deployment gates (Azure Monitor, REST API, Azure Policy, invoke function). GHA environments only support required reviewers, wait timers, and custom deployment protection rules.

**Workaround:** Use GitHub Apps deployment protection rules (custom webhook gates) or separate validation jobs.
**Complexity:** Medium-High

```yaml
jobs:
  pre-deploy-validation:
    runs-on: ubuntu-latest
    steps:
      - name: Check Azure Monitor alerts
        uses: azure/CLI@v2
        with:
          inlineScript: |
            alerts=$(az monitor alert list -g my-rg --query "[?properties.status=='Activated']" -o tsv)
            if [ -n "$alerts" ]; then echo "Active alerts found" && exit 1; fi

      - name: Check API health
        run: |
          status=$(curl -s -o /dev/null -w "%{http_code}" https://dependency-api.example.com/health)
          if [ "$status" != "200" ]; then exit 1; fi

  deploy:
    needs: pre-deploy-validation
    environment: production          # Also gets manual approval via protection rules
    runs-on: ubuntu-latest
    steps:
      - uses: azure/webapps-deploy@v3
        # ...deployment steps

  post-deploy-validation:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: Smoke tests
        run: ./scripts/smoke-test.sh https://myapp.azurewebsites.net
```

For true gate functionality, implement a **custom GitHub App** that responds to the `deployment_protection_rule` webhook event and calls external APIs before approving.

---

## 3. Pipeline Decorators

**Gap:** ADO pipeline decorators inject steps into all pipelines organization-wide (e.g., security scanning, compliance logging). GHA has no equivalent injection mechanism.

**Workaround:** Use required workflows (organization-level) or mandatory composite actions.
**Complexity:** Low-Medium

- **Required workflows** (GitHub Enterprise Cloud): Organization admins can require specific workflows to pass on all repos
- **Shared composite actions:** Publish a composite action teams must call; enforce via required status checks
- **Starter workflows:** Provide templates in `.github` org repo (not enforced, but discoverable)

```yaml
# Org-level required workflow (.github/workflows/required-security-scan.yml)
name: Required Security Scan
on:
  pull_request:
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
      - uses: github/codeql-action/analyze@v3
```

---

## 4. Variable Group → Key Vault Link

**Gap:** ADO variable groups can link directly to Azure Key Vault, automatically syncing secrets. GHA has no native Key Vault variable group concept.

**Workaround:** Use `azure/login` + `azure/get-keyvault-secrets` action pattern.
**Complexity:** Low

```yaml
steps:
  - uses: azure/login@v2
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

  - uses: azure/get-keyvault-secrets@v1
    with:
      keyvault: 'my-keyvault'
      secrets: 'db-connection-string, api-key, storage-account-key'
    id: kv-secrets

  - name: Use secrets
    run: echo "Connecting to DB..."
    env:
      DB_CONN: ${{ steps.kv-secrets.outputs.db-connection-string }}
      API_KEY: ${{ steps.kv-secrets.outputs.api-key }}
```

---

## 5. Secure Files

**Gap:** ADO Secure Files stores binary files (certificates, provisioning profiles, keystores) for pipeline use. GHA has no equivalent.

**Workaround:** Base64-encode files into secrets, or retrieve from Key Vault.
**Complexity:** Low

```yaml
# Option A: Base64-encoded secret
steps:
  - name: Decode signing certificate
    run: |
      echo "${{ secrets.SIGNING_CERT_BASE64 }}" | base64 -d > signing-cert.pfx

# Option B: Key Vault certificate retrieval
  - uses: azure/CLI@v2
    with:
      inlineScript: |
        az keyvault certificate download --vault-name my-kv \
          --name signing-cert --file signing-cert.pfx --encoding PEM
```

**To encode:** `base64 -w 0 < cert.pfx | pbcopy` (macOS) or `[Convert]::ToBase64String([IO.File]::ReadAllBytes("cert.pfx"))` (PowerShell). Store the output as a GitHub secret.

---

## 6. Runtime Expressions $[ ]

**Gap:** ADO runtime expressions (`$[ ]`) evaluate at run time within a job, enabling dynamic variable setting based on prior step outputs. GHA uses different conditional mechanisms.

**Workaround:** Restructure using job-level `if:` conditions, step outputs, or environment files.
**Complexity:** Low

```yaml
# ADO: $[ eq(variables['Build.SourceBranch'], 'refs/heads/main') ]
# GHA equivalent patterns:

# Pattern 1: Step-level conditional
- name: Deploy
  if: github.ref == 'refs/heads/main'
  run: ./deploy.sh

# Pattern 2: Dynamic variable via GITHUB_OUTPUT
- name: Set config
  id: config
  run: |
    if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
      echo "environment=production" >> $GITHUB_OUTPUT
    else
      echo "environment=staging" >> $GITHUB_OUTPUT
    fi

- name: Deploy to ${{ steps.config.outputs.environment }}
  run: ./deploy.sh ${{ steps.config.outputs.environment }}
```

---

## 7. Resource-Level Checks

**Gap:** ADO supports checks on service connections, agent pools, variable groups, and environments. GHA protection rules only apply to environments.

**Workaround:** Consolidate all resource checks into environment protection rules.
**Complexity:** Low-Medium

- Assign each deployment target (Azure sub, K8s cluster) its own GitHub environment
- Attach protection rules (approvals, wait timers, branch restrictions) to the environment
- Use OIDC scoped to each environment for Azure access control
- Implement custom deployment protection rules (GitHub Apps) for advanced checks

---

## 8. TFVC Support

**Gap:** GHA does not support TFVC repositories. ADO Pipelines can build from TFVC.

**Workaround:** Convert TFVC to Git before migration using git-tfs.
**Complexity:** Medium-High (depends on repo size and history)

```bash
# Convert TFVC to Git
git tfs clone https://dev.azure.com/{org} $/Project/Main --branches=all
cd Main
git lfs migrate import --include="*.dll,*.exe,*.zip" --everything
git remote add github https://github.com/{org}/{repo}.git
git push github --mirror
```

See `repo-migration-strategy.md` for detailed TFVC conversion guidance.

---

## 9. Task Groups (Classic Pipelines)

**Gap:** ADO classic task groups bundle reusable step sequences with parameters. GHA has no classic editor.

**Workaround:** Convert to composite actions with inputs/outputs.
**Complexity:** Low

```yaml
# .github/actions/build-dotnet/action.yml (composite action)
name: 'Build .NET App'
inputs:
  project-path:
    required: true
  configuration:
    default: 'Release'
runs:
  using: composite
  steps:
    - run: dotnet restore ${{ inputs.project-path }}
      shell: bash
    - run: dotnet build ${{ inputs.project-path }} -c ${{ inputs.configuration }}
      shell: bash
    - run: dotnet test ${{ inputs.project-path }} -c ${{ inputs.configuration }} --no-build
      shell: bash
```

---

## 10. Pipeline Analytics & Reports

**Gap:** ADO provides built-in pipeline analytics (pass rate, duration trends, failure analysis). GHA has no native analytics dashboard.

**Workaround:** Use GitHub API, third-party dashboards, or custom reporting.
**Complexity:** Medium

- **GitHub Actions Usage API** — query workflow run data programmatically
- **Datadog CI Visibility** — ingest GHA workflow telemetry for dashboards
- **Grafana + GitHub Exporter** — custom dashboards via Prometheus exporter
- **GITHUB_STEP_SUMMARY** — per-run summaries with Markdown formatting
- **Custom reporting workflow** — scheduled workflow that queries API and posts results

---

## 11. Azure Test Plans Integration

**Gap:** ADO Azure Test Plans provides manual test management, test suites, and exploratory testing. No GitHub equivalent exists.

**Workaround:** Keep ADO Test Plans (hybrid) or migrate to third-party.
**Complexity:** Low (hybrid) / Medium-High (third-party migration)

- **Hybrid:** Continue using Azure Test Plans alongside GitHub repos; link test cases to GitHub issues via URLs
- **TestRail:** Import test cases via API; supports GitHub integration
- **Zephyr Scale:** Jira-ecosystem; suitable if also migrating project management
- **GitHub Issues:** Lightweight option using issue templates for test tracking (limited)

---

## 12. Audit Log Differences

**Gap:** ADO and GitHub have different audit log schemas and event coverage.

**Workaround:** Map ADO audit events to GitHub audit log equivalents; supplement with workflow logging.
**Complexity:** Low

| ADO Audit Event | GitHub Audit Log Equivalent |
|---|---|
| Pipeline run created | `workflows.completed_workflow_run` |
| Service connection used | OIDC token issued (in workflow logs) |
| Variable group accessed | Secret access (in workflow logs) |
| Approval granted | `environment.create_actions_secret` (partial) |
| Branch policy changed | `protected_branch.update` or `repository_ruleset.update` |
| Repository access changed | `repo.access` |

For compliance, enable **audit log streaming** to a SIEM (Splunk, Sentinel, Datadog) for both ADO and GitHub during transition. GitHub Enterprise Cloud supports streaming to Azure Event Hubs, Splunk, Datadog, and S3.

---

## Summary: Gap Severity Matrix

| Gap | Impact | Workaround Complexity | Recommendation |
|---|---|---|---|
| Deployment strategies | Medium | Medium | Use infra-level strategies |
| Pre/post gates | High | Medium-High | Validation jobs + custom GitHub Apps |
| Pipeline decorators | Medium | Low-Medium | Required workflows (Enterprise) |
| Variable group → KV | Low | Low | azure/get-keyvault-secrets action |
| Secure files | Low | Low | Base64 secrets or KV certs |
| Runtime expressions | Low | Low | Step outputs + if: conditions |
| Resource-level checks | Medium | Low-Medium | Consolidate to environment rules |
| TFVC support | High | Medium-High | git-tfs conversion (one-time) |
| Task groups | Low | Low | Composite actions |
| Pipeline analytics | Medium | Medium | Third-party or API-based |
| Azure Test Plans | Low-Medium | Low-High | Hybrid recommended |
| Audit logs | Low | Low | Map events + streaming |
