# OIDC Federated Identity for Cloud Deployments — Deep Reference

## How OIDC Works in GitHub Actions

1. Workflow requests a short-lived JWT from GitHub's OIDC provider (`token.actions.githubusercontent.com`)
2. The JWT contains claims about the workflow (repo, branch, environment, etc.)
3. Cloud provider validates the JWT against its trust policy
4. If claims match, cloud provider issues temporary credentials
5. **No stored secrets needed** — eliminates secret rotation, reduces blast radius

Required permission in the workflow:

```yaml
permissions:
  id-token: write   # Required to request the OIDC JWT
  contents: read    # Required for actions/checkout
```

## OIDC Claim Reference

### Subject (`sub`) Claim Formats

| Trigger | Subject Format |
|---------|---------------|
| Branch push | `repo:owner/repo:ref:refs/heads/branch-name` |
| Tag push | `repo:owner/repo:ref:refs/tags/tag-name` |
| Environment | `repo:owner/repo:environment:env-name` |
| Pull request | `repo:owner/repo:pull_request` |
| Reusable workflow (caller) | `repo:owner/caller-repo:ref:refs/heads/main` |
| Reusable workflow (job) | `repo:owner/reusable-repo:ref:refs/heads/main` |

### Other Key Claims

| Claim | Example Value |
|-------|--------------|
| `iss` | `https://token.actions.githubusercontent.com` |
| `aud` | `api://AzureADTokenExchange` (Azure) or custom |
| `repository` | `my-org/my-repo` |
| `repository_owner` | `my-org` |
| `ref` | `refs/heads/main` |
| `environment` | `production` |
| `job_workflow_ref` | `my-org/reusable/.github/workflows/deploy.yml@refs/heads/main` |
| `runner_environment` | `github-hosted` |

## Azure Setup (Entra ID + Federated Credentials)

### Step 1: Create App Registration

```bash
# Create the Entra ID app registration
az ad app create --display-name "github-actions-deploy"
APP_ID=$(az ad app list --display-name "github-actions-deploy" --query "[0].appId" -o tsv)

# Create a service principal
az ad sp create --id $APP_ID

# Assign RBAC role
az role assignment create \
  --assignee $APP_ID \
  --role "Contributor" \
  --scope "/subscriptions/<subscription-id>/resourceGroups/<rg-name>"
```

### Step 2: Add Federated Credential

```bash
# Trust GitHub OIDC for main branch
az ad app federated-credential create --id $APP_ID --parameters '{
  "name": "github-main-branch",
  "issuer": "https://token.actions.githubusercontent.com",
  "subject": "repo:my-org/my-repo:ref:refs/heads/main",
  "audiences": ["api://AzureADTokenExchange"],
  "description": "Deploy from main branch"
}'

# Trust GitHub OIDC for production environment
az ad app federated-credential create --id $APP_ID --parameters '{
  "name": "github-production-env",
  "issuer": "https://token.actions.githubusercontent.com",
  "subject": "repo:my-org/my-repo:environment:production",
  "audiences": ["api://AzureADTokenExchange"],
  "description": "Deploy to production environment"
}'
```

### Step 3: Workflow Configuration

```yaml
name: Deploy to Azure
on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}

      - uses: azure/webapps-deploy@v3
        with:
          app-name: my-web-app
          package: ./dist
```

## AWS Setup (IAM OIDC Provider + Role)

### Step 1: Create OIDC Provider and IAM Role

```bash
# Create the OIDC provider
aws iam create-open-id-connect-provider \
  --url "https://token.actions.githubusercontent.com" \
  --thumbprint-list "ffffffffffffffffffffffffffffffffffffffff" \
  --client-id-list "sts.amazonaws.com"
```

### Step 2: IAM Trust Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
    },
    "Action": "sts:AssumeRoleWithWebIdentity",
    "Condition": {
      "StringEquals": {
        "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
        "token.actions.githubusercontent.com:sub": "repo:my-org/my-repo:ref:refs/heads/main"
      }
    }
  }]
}
```

### Step 3: Workflow Configuration

```yaml
name: Deploy to AWS
on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::ACCOUNT_ID:role/github-actions-role
          aws-region: us-east-1

      - run: aws s3 sync ./dist s3://my-bucket
```

## GCP Setup (Workload Identity Federation)

### Step 1: Create Workload Identity Pool and Provider

```bash
# Create workload identity pool
gcloud iam workload-identity-pools create "github-pool" \
  --location="global" \
  --display-name="GitHub Actions Pool"

# Create provider
gcloud iam workload-identity-pools providers create-oidc "github-provider" \
  --location="global" \
  --workload-identity-pool="github-pool" \
  --display-name="GitHub Provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com"

# Bind service account
gcloud iam service-accounts add-iam-policy-binding "deploy@project-id.iam.gserviceaccount.com" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/PROJECT_NUM/locations/global/workloadIdentityPools/github-pool/attribute.repository/my-org/my-repo"
```

### Step 2: Workflow Configuration

```yaml
name: Deploy to GCP
on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: 'projects/PROJECT_NUM/locations/global/workloadIdentityPools/github-pool/providers/github-provider'
          service_account: 'deploy@project-id.iam.gserviceaccount.com'

      - uses: google-github-actions/deploy-cloudrun@v2
        with:
          service: my-service
          region: us-central1
          image: gcr.io/project-id/my-app:${{ github.sha }}
```

## Environment-Scoped OIDC (Multi-Environment)

Use separate federated credentials per environment — each trusts only its specific environment claim:

```yaml
jobs:
  deploy-dev:
    runs-on: ubuntu-latest
    environment: development
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID_DEV }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID_DEV }}

  deploy-prod:
    needs: deploy-dev
    runs-on: ubuntu-latest
    environment: production
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID_PROD }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID_PROD }}
```

Each environment's federated credential trusts ONLY its own subject:
- Dev: `repo:my-org/my-repo:environment:development`
- Prod: `repo:my-org/my-repo:environment:production`

## Trust Policy Best Practices

**NEVER use wildcard trust** — constrain by repo + branch or repo + environment:

```json
// GOOD — constrained to specific repo and branch
"sub": "repo:my-org/my-repo:ref:refs/heads/main"

// GOOD — constrained to specific repo and environment
"sub": "repo:my-org/my-repo:environment:production"

// DANGEROUS — allows any branch in the repo
"sub": "repo:my-org/my-repo:*"

// CATASTROPHIC — allows any repo in the org
"sub": "repo:my-org/*"
```

## Multiple Cloud Providers in One Workflow

```yaml
jobs:
  deploy-all:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ vars.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: ${{ vars.GCP_WORKLOAD_IDENTITY }}
          service_account: ${{ vars.GCP_SERVICE_ACCOUNT }}
```

## Troubleshooting OIDC

| Error | Cause | Fix |
|-------|-------|-----|
| `AADSTS70021: No matching federated identity record found` | Subject claim mismatch | Verify branch/environment name matches federated credential exactly |
| `Audience mismatch` | Wrong `aud` claim | Azure expects `api://AzureADTokenExchange`, AWS expects `sts.amazonaws.com` |
| `Error: Not authorized to perform sts:AssumeRoleWithWebIdentity` | IAM trust policy mismatch | Check `sub` and `aud` conditions in the trust policy |
| `id-token permission not set` | Missing workflow permission | Add `permissions: id-token: write` at workflow or job level |
| `Could not get ID Token` | Missing `id-token: write` or wrong workflow level | Ensure `permissions` block exists and isn't overridden |

## Migration from Stored Secrets to OIDC

1. **Create** the cloud identity (app registration/IAM role/workload identity)
2. **Add** federated credentials trusting your repo + branch/environment
3. **Store** the non-secret identifiers as repository **variables** (not secrets) — client-id, tenant-id, subscription-id, role ARN
4. **Update** the workflow to use the OIDC login action instead of credential secrets
5. **Test** on a feature branch (add a federated credential for the test branch)
6. **Remove** the old stored credential secrets after confirming OIDC works
7. **Delete** the temporary test branch federated credential
