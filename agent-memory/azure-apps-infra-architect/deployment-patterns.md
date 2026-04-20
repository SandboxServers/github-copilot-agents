# Deployment Patterns

## Blue-Green (App Service Slots)

```bash
# Deploy to staging slot
az webapp deployment source config-zip -g rg -n myapp --src app.zip --slot staging

# Verify staging
curl https://myapp-staging.azurewebsites.net/health

# Swap staging → production (zero downtime)
az webapp deployment slot swap -g rg -n myapp --slot staging --target-slot production

# Rollback if needed — swap again
az webapp deployment slot swap -g rg -n myapp --slot staging --target-slot production
```

Slot settings (sticky): ASPNETCORE_ENVIRONMENT, connection strings marked as "slot setting". These don't swap.

## Canary (Container Apps)

```bash
# Deploy new revision
az containerapp update -g rg -n myapp --image myapp:v2 --revision-suffix v2

# Send 10% traffic to new revision
az containerapp ingress traffic set -g rg -n myapp \
  --revision-weight myapp--v1=90 myapp--v2=10

# Gradually increase to 50%, then 100%
az containerapp ingress traffic set -g rg -n myapp \
  --revision-weight myapp--v2=100
```

## Rolling Updates (AKS)

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
```

Requires readiness and liveness probes. Pod disruption budgets for availability.

## IaC Deployment

Bicep (preferred by Microsoft), Terraform (multi-cloud). ARM templates (legacy). Deployment stacks for lifecycle management. What-if before apply.

## CI/CD with OIDC

GitHub Actions: workload identity federation. No secrets stored. azure/login action with client-id, tenant-id, subscription-id.
Azure Pipelines: Workload identity federation service connections. ARM_USE_OIDC=true.
