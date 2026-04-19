## Workload Identity Federation

### The Problem It Solves
External workloads (GitHub Actions, Kubernetes, other clouds) historically needed client secrets or certificates to authenticate to Entra ID. These secrets:
- Expire and cause outages
- Can be leaked in logs, repos, configs
- Must be rotated, creating operational burden

### How It Works
```
1. External IdP (GitHub, GCP, AWS, Kubernetes) issues a token to the workload
2. Workload presents that token to Microsoft identity platform
3. Entra ID validates the token against the configured trust relationship
4. Entra ID issues an access token — no secrets exchanged
```

### Supported Scenarios
| External IdP | Common Use Case |
|--------------|----------------|
| **GitHub Actions** | CI/CD deploying to Azure without secrets |
| **Google Cloud** | GCP workloads accessing Azure resources |
| **AWS** | Cross-cloud access scenarios |
| **Kubernetes (any)** | Pods accessing Azure resources via projected service account tokens |
| **Other OIDC providers** | Any IdP that issues OIDC tokens with standard claims |

### Configuration
- Create federated credential on an **app registration** or **user-assigned managed identity**
- Define: issuer URI, subject identifier, audience
- For GitHub Actions: issuer = `https://token.actions.githubusercontent.com`, subject = `repo:org/repo:ref:refs/heads/main`
- Max 20 federated credentials per app registration
