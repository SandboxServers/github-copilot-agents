## Workload Identity

### The Problem with Secrets

External workloads (CI/CD, Kubernetes, other clouds) historically used client secrets or certificates to authenticate to Azure. Secrets create risk:
- Expire and cause production outages
- Can be leaked in logs, repos, or config files
- Must be rotated on schedule, creating operational burden
- Stored in multiple places, hard to track

### Workload Identity Federation

Federated credentials allow external OIDC providers to authenticate to Entra ID without secrets:

```
1. External IdP (GitHub, Kubernetes, GCP) issues an OIDC token to the workload
2. Workload presents that token to Microsoft identity platform
3. Entra ID validates the token against the configured trust relationship
4. Entra ID issues an access token — no secrets exchanged
```

### Trust Policy Configuration

A federated identity credential defines the trust:
- **Issuer**: OIDC issuer URL of the external IdP
- **Subject**: identifier of the specific workload (repo, namespace, service account)
- **Audience**: expected audience claim (typically `api://AzureADTokenExchange`)
- Configured on an app registration or user-assigned managed identity
- Max 20 federated credentials per app registration

### Managed Identity (Azure Resources)

For workloads running IN Azure, managed identity is the simplest option:

| Type | Lifecycle | Sharing | Use Case |
|------|-----------|---------|----------|
| **System-assigned** | Tied to resource | Cannot be shared | Single resource, simple scenarios |
| **User-assigned** | Independent | Shared across resources | Multiple resources, same identity |

- No credentials to manage — Azure handles token issuance automatically
- Supported on: VMs, App Service, Functions, AKS, Container Apps, Logic Apps, and more
- Assign RBAC roles directly to the managed identity

### Supported Federation Scenarios

| External IdP | Issuer | Subject Format |
|--------------|--------|---------------|
| **GitHub Actions** | `https://token.actions.githubusercontent.com` | `repo:org/repo:ref:refs/heads/main` |
| **AKS Workload Identity** | OIDC issuer URL of the AKS cluster | `system:serviceaccount:namespace:sa-name` |
| **Terraform Cloud** | `https://app.terraform.io` | `organization:org:project:proj:workspace:ws:run_phase:plan` |
| **Google Cloud** | `https://accounts.google.com` | Subject from GCP service account |
| **AWS** | STS regional endpoint | Role ARN |

### GitHub Actions OIDC Example

```yaml
permissions:
  id-token: write    # Required for OIDC
  contents: read

steps:
  - uses: azure/login@v2
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

### Decision Tree

```
Workload runs IN Azure?
  → Use managed identity (system-assigned or user-assigned)

Workload runs OUTSIDE Azure with OIDC support?
  → Use workload identity federation (no secrets)

Workload has no OIDC support?
  → Use certificate credential (not secret)
  → Set short expiry, monitor, rotate

Never: long-lived client secrets in production
```
