# Azure Pipelines Security Hardening

## Service Connection Security

Service connections grant pipelines access to external systems (Azure, AWS, Docker, Kubernetes). Apply least-privilege principles rigorously.

```yaml
# Scope service connections to specific environments via checks
# Configure in: Project Settings > Service connections > [connection] > Approvals and checks

# YAML — reference a service connection
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'prod-deploy-sc'   # Service connection name
    appName: my-app
```

**Best practices:**
- Create separate service connections per environment (dev-sc, staging-sc, prod-sc)
- Scope Azure service principals to specific resource groups, not subscriptions
- Enable **Restrict pipeline access** so only explicitly authorized pipelines can use the connection
- Add **Approval checks** on production service connections requiring manual sign-off
- Add **Branch control** checks to restrict production connections to `main`/`release/*` only

## Protected Resources and Checks

Protected resources include: service connections, environments, repositories, agent pools, variable groups (with secrets), and secure files.

```yaml
# Environment with approval gate — deployment pauses until approved
stages:
  - stage: Production
    jobs:
      - deployment: DeployProd
        environment: production       # Protected resource
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo deploying
```

Available checks on protected resources:
- **Approvals**: Manual approval by specified users/groups
- **Branch control**: Only allow pipelines running from specific branches
- **Business hours**: Restrict deployments to specific time windows
- **Required template**: Enforce that pipelines use a specific YAML template
- **Exclusive lock**: Only one pipeline run can use the resource at a time
- **Invoke Azure Function / REST API**: Custom validation logic

## Pipeline Permissions

By default, new protected resources require explicit pipeline authorization.

- **Never enable Open Access** on production resources — it allows all pipelines in the project
- Grant access pipeline-by-pipeline via: Resource > Security > Pipeline permissions
- Revoking access is immediate; the pipeline will fail on next run

## Secure Files

Upload certificates, provisioning profiles, and SSH keys via Library > Secure files.

```yaml
steps:
  - task: DownloadSecureFile@1
    name: sslCert
    inputs:
      secureFile: 'prod-certificate.pfx'
  - script: |
      echo "Cert downloaded to: $(sslCert.secureFilePath)"
      # Use the file — it's automatically cleaned up after the job
```

Secure files are encrypted at rest and only downloaded to the agent during the job. They are deleted after the pipeline job completes.

## Secrets Management Hierarchy

| Method | Scope | Rotation | Best For |
|--------|-------|----------|----------|
| YAML `variables` with `secret: true` | Pipeline | Manual | Quick prototyping only |
| Variable groups (secret variables) | Project | Manual | Shared secrets across pipelines |
| Variable group linked to Key Vault | Project + KV | Automatic via KV | Production secrets |
| `AzureKeyVault@2` task | Per-job | Automatic | Runtime secret fetching |

```yaml
# Key Vault linked variable group — secrets auto-sync
variables:
  - group: prod-secrets-from-kv     # Linked to Azure Key Vault

# Direct Key Vault task — fetches at runtime
steps:
  - task: AzureKeyVault@2
    inputs:
      azureSubscription: 'my-sc'
      KeyVaultName: 'my-prod-kv'
      SecretsFilter: 'SqlPassword,ApiKey'   # Comma-separated, or '*' for all
      RunAsPreJob: true                      # Available to all subsequent steps
```

**Secret safety rules:**
- Secrets are automatically masked in logs (replaced with `***`)
- Never use `echo $(secret)` — even masked, it confirms the variable exists
- Secrets are NOT available in template expressions (`${{ }}` compile-time context)
- Secrets cannot be passed to downstream jobs/stages without explicit output variable mapping

## Pipeline Decorators

Organization-level injected steps that run in every pipeline — ideal for mandatory security scanning.

- Defined as YAML templates installed as Azure DevOps extensions
- Inject steps before or after all jobs, or target specific tasks
- Cannot be bypassed by pipeline authors
- Use for: container scanning, credential scanning, license compliance, SBOM generation

## Forked Repository Build Security

Builds from forked repositories pose unique risks — the fork author controls the YAML.

- **Do not expose secrets to fork builds** — disabled by default for good reason
- PR validation from forks runs with reduced permissions
- Require a team member to comment `/azp run` before building fork PRs
- Consider requiring fork PRs to target a `contrib` branch, not `main`

## Template-Required Checks

Enforce that pipelines consuming a protected resource must `extend` from an approved template:

```yaml
# Required template: security-baseline.yml
# Configured on: Environment > Approvals and checks > Required template
# All pipelines deploying to this environment MUST use this template

# security-baseline.yml (in shared repo)
parameters:
  - name: stages
    type: stageList
stages:
  - stage: SecurityScan
    jobs:
      - job: Scan
        steps:
          - script: run-security-scan.sh
  - ${{ each stage in parameters.stages }}:
    - ${{ stage }}
```

## Audit and Compliance

- Azure DevOps Audit Logs capture: pipeline creation/modification, service connection access, variable group changes, approval actions
- Access audit logs via: Organization Settings > Auditing
- Stream audit logs to Azure Monitor, Splunk, or a SIEM for retention beyond 90 days
- Use the Auditing REST API for automated compliance checks

## YAML Modification Controls

- Protect the pipeline YAML file via branch policies (require PR + reviewer for YAML changes)
- Use `extends` templates to enforce structure — pipeline authors fill parameters, not raw YAML
- Enable "Limit job authorization scope to referenced Azure DevOps repositories"
- Enable "Limit job authorization scope to current project" in organization settings
