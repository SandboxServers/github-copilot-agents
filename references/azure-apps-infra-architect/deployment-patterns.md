# Deployment Patterns

## Deployment Method Selection

| Method | Best For | Gotchas |
|--------|----------|---------|
| Azure DevOps Pipelines | Enterprise with ADO ecosystem | YAML complexity, marketplace dependency |
| GitHub Actions | GitHub-native teams | Action quality varies, marketplace trust |
| Terraform + CI/CD | Multi-cloud, complex infra | State management, provider lag |
| Bicep + deployment stacks | Azure-native, landing zones | Azure-only, newer ecosystem |
| Azure CLI/PowerShell | Quick automation, scripts | Not idempotent by default |

## Zero-Downtime Deployment

- **App Service**: Deployment slots → swap. Always use staging slot + swap. Never deploy directly to production slot.
- **Container Apps**: Revision-based traffic splitting. Deploy new revision, shift traffic gradually (10% → 50% → 100%).
- **AKS**: Rolling update (default), blue-green via service routing, canary via Flagger/Argo Rollouts.
- **Functions**: Deployment slots (Premium plan only). Consumption plan: deploy and pray (usually fine due to fast cold starts).
