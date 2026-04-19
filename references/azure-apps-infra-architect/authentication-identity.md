# Authentication & Identity Patterns

## Managed Identity Decision Tree

```
Does the Azure service support managed identity?
├─ Yes → Use managed identity. Period.
│        ├─ Single resource with unique permissions? → System-assigned
│        ├─ Multiple resources sharing permissions? → User-assigned
│        └─ IaC deployment (need identity before resource)? → User-assigned
└─ No → Does the external service support Entra ID auth?
         ├─ Yes → App registration with federated credential (no secrets)
         └─ No → Key Vault for secret storage, managed identity to access KV
```

## App Registration vs Enterprise App

- **App registration**: The application's identity definition. You create this. It defines what the app is and what it can request.
- **Enterprise app (service principal)**: The instance of that app in a tenant. Created automatically when you register or consent. It defines what the app actually gets in this tenant.
- **Rule**: One app registration, one enterprise app per tenant it's used in.
- **Modern pattern**: Workload identity federation for CI/CD (GitHub Actions → Entra without secrets). Federated credentials on the app registration.
- **Anti-pattern**: Client secrets on app registrations. They expire, they leak, they're shared. Use certificates or federated credentials instead.
- **Anti-pattern**: App registrations with broad permissions (`Directory.ReadWrite.All`) that nobody remembers why.

## Auth Flow Selection

| Scenario | Flow | Notes |
|----------|------|-------|
| Web app with user sign-in | Authorization Code + PKCE | Default for modern web apps |
| SPA | Authorization Code + PKCE | Implicit flow is deprecated |
| API-to-API (no user) | Client Credentials | Use certificate, not secret |
| CI/CD pipeline | Workload Identity Federation | No secrets stored anywhere |
| Azure resource to Azure resource | Managed Identity | Always the answer |
| Daemon/service on-prem | Client Credentials + certificate | Key Vault for cert if possible |
| B2B partner access | Entra External ID (B2B) | Guest accounts with conditional access |
| Customer-facing auth | Entra External ID (CIAM) | Replaced Azure AD B2C |
