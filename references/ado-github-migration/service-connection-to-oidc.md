# Service Connection to OIDC Migration

## Why OIDC Over Stored Secrets

ADO service connections typically store long-lived credentials (client secrets, PATs, keys).
GitHub Actions supports OpenID Connect (OIDC) federated credentials — short-lived tokens
issued per workflow run with no stored secrets.

**Benefits:**
- **No secret rotation** — tokens are ephemeral (valid ~10 minutes)
- **Auditable** — every token issuance is logged with repo, branch, and workflow context
- **Zero-trust aligned** — credentials are scoped to specific repos, branches, and environments
- **No secret sprawl** — nothing stored in GitHub that can be leaked or exfiltrated

## Service Connection Type Migration Map

### Azure Resource Manager → OIDC with azure/login

**ADO (most common migration):**
```yaml
# ADO service connection reference
- task: AzureCLI@2
  inputs:
    azureSubscription: 'my-azure-connection'
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      az webapp deploy --name myapp --src-path ./app.zip
```

**GitHub Actions with OIDC:**
```yaml
permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # Ties to federated credential subject
    steps:
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - uses: azure/CLI@v2
        with:
          inlineScript: |
            az webapp deploy --name myapp --src-path ./app.zip
```

### AWS Service Connection → OIDC with aws-actions/configure-aws-credentials

```yaml
permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/my-github-role
          aws-region: us-east-1
      - run: aws s3 sync ./dist s3://my-bucket
```

### Docker Registry → docker/login-action

```yaml
# Azure Container Registry with OIDC
- uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
- uses: docker/login-action@v3
  with:
    registry: myregistry.azurecr.io
    # Token auto-populated after azure/login

# Docker Hub with stored credentials
- uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

### Kubernetes Service Connection → azure/k8s-set-context

```yaml
- uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
- uses: azure/aks-set-context@v4
  with:
    resource-group: my-rg
    cluster-name: my-aks
```

### Generic / REST API → Repository Secrets or GitHub App Token

No OIDC equivalent — store API keys as GitHub secrets:
```yaml
- name: Call external API
  run: |
    curl -H "Authorization: Bearer ${{ secrets.API_TOKEN }}" \
      https://api.example.com/deploy
```

### GitHub Service Connection → GITHUB_TOKEN (Automatic)

ADO GitHub service connections become unnecessary — `GITHUB_TOKEN` is automatic:
```yaml
- name: Create release
  uses: actions/create-release@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Azure OIDC Setup Step-by-Step

### 1. Create App Registration

```bash
# Create the app registration
az ad app create --display-name "github-actions-myrepo"

# Note the appId from output
APP_ID=$(az ad app create --display-name "github-actions-myrepo" --query appId -o tsv)

# Create service principal
az ad sp create --id $APP_ID
```

### 2. Configure Federated Credential

```bash
# Create federated credential for a specific environment
az ad app federated-credential create --id $APP_ID --parameters '{
  "name": "github-prod-environment",
  "issuer": "https://token.actions.githubusercontent.com",
  "subject": "repo:my-org/my-repo:environment:production",
  "description": "GitHub Actions production deployments",
  "audiences": ["api://AzureADTokenExchange"]
}'
```

### 3. Assign RBAC Role

```bash
az role assignment create \
  --assignee $APP_ID \
  --role "Contributor" \
  --scope /subscriptions/$SUBSCRIPTION_ID/resourceGroups/my-rg
```

### 4. Store IDs as GitHub Secrets

```bash
gh secret set AZURE_CLIENT_ID --body "$APP_ID"
gh secret set AZURE_TENANT_ID --body "$TENANT_ID"
gh secret set AZURE_SUBSCRIPTION_ID --body "$SUBSCRIPTION_ID"
```

## Federated Credential Subject Formats

The `subject` field in the federated credential controls which workflows can use it:

| Scenario | Subject Format | Use Case |
|---|---|---|
| Specific branch | `repo:owner/repo:ref:refs/heads/main` | Production deploys from main |
| Any branch | `repo:owner/repo:ref:refs/heads/*` | CI builds on any branch |
| Environment | `repo:owner/repo:environment:production` | **Recommended** for deployments |
| Pull request | `repo:owner/repo:pull_request` | PR validation (read-only roles) |
| Tag | `repo:owner/repo:ref:refs/tags/v*` | Release deployments |

**Best practice:** Use `environment:` subjects for deployments. This ties OIDC trust to GitHub
environment protection rules (approvals, branch restrictions), providing defense-in-depth.

## Multi-Environment Setup

Create separate credentials per environment with different trust policies and RBAC:

```bash
# Dev — broader access, fewer restrictions
az ad app federated-credential create --id $DEV_APP_ID --parameters '{
  "name": "github-dev",
  "issuer": "https://token.actions.githubusercontent.com",
  "subject": "repo:my-org/my-repo:environment:dev",
  "audiences": ["api://AzureADTokenExchange"]
}'
az role assignment create --assignee $DEV_APP_ID --role "Contributor" \
  --scope /subscriptions/$SUB_ID/resourceGroups/rg-dev

# Prod — locked down, environment approval required
az ad app federated-credential create --id $PROD_APP_ID --parameters '{
  "name": "github-prod",
  "issuer": "https://token.actions.githubusercontent.com",
  "subject": "repo:my-org/my-repo:environment:production",
  "audiences": ["api://AzureADTokenExchange"]
}'
az role assignment create --assignee $PROD_APP_ID --role "Contributor" \
  --scope /subscriptions/$SUB_ID/resourceGroups/rg-prod
```

## Service Connection Inventory via REST API

Audit all ADO service connections before migration:

```bash
# List all service endpoints in a project
curl -u :$ADO_PAT \
  "https://dev.azure.com/{org}/{project}/_apis/serviceendpoint/endpoints?api-version=7.1" \
  | jq '.value[] | {name, type, url, authorization: .authorization.scheme}'
```

Produces output like:
```json
{ "name": "Azure Production", "type": "azurerm", "url": "https://management.azure.com/", "authorization": "ServicePrincipal" }
{ "name": "Docker Hub", "type": "dockerregistry", "url": "https://index.docker.io/v1/", "authorization": "UsernamePassword" }
{ "name": "GitHub", "type": "github", "url": "https://github.com", "authorization": "Token" }
```

## Migration Validation Checklist

1. **Create OIDC credential** → Verify with `az ad app federated-credential list`
2. **Test login** → Run a minimal workflow that does `azure/login` + `az account show`
3. **Verify RBAC** → Ensure the SP has correct roles on target resources
4. **Test deploy** → Run actual deployment to a non-production environment
5. **Audit logs** → Check Entra ID sign-in logs for successful OIDC authentications
6. **Burn-in period** → Keep ADO service connections active for 2-4 weeks as fallback
7. **Decommission** → Delete ADO service connections after confirmed stability

## Edge Cases

- **Managed identity backend**: ADO service connections using managed identity (no client secret)
  cannot directly migrate — create a new app registration with federated credentials instead
- **Multi-tenant**: For cross-tenant deployments, configure federated credentials in each tenant
  and use separate `azure/login` steps with different tenant IDs
- **Sovereign clouds**: Set the `environment` input on `azure/login` (e.g., `AzureUSGovernment`,
  `AzureChinaCloud`) and update the audience in federated credentials accordingly
- **Key Vault access**: If the ADO service connection accesses Key Vault, ensure the new SP
  has `Key Vault Secrets User` role — RBAC-based access is preferred over access policies
