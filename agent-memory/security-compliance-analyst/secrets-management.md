# Secrets Management

The hierarchy of authentication methods, from most secure to least:

## Authentication Hierarchy

| Rank | Method | Security | Notes |
|---|---|---|---|
| **1. Best** | Managed identity | No secrets exist | Nothing to leak, rotate, or manage. Always prefer this. |
| **2. Good** | Certificate credential | Rotatable, harder to exfiltrate | Stored in Key Vault, can be automated |
| **3. Acceptable** | Key Vault secret | Centralized, audited, access-controlled | Requires access management, rotation policy |
| **4. Bad** | App setting / environment variable | Not audited, visible in portal | Secrets exposed in deployment logs, portal, config |
| **5. Unacceptable** | Source code | Committed to repo, in git history forever | Immediate Critical finding. Rotate secret immediately. |

## Azure Key Vault

Key Vault is the centralized secrets management service:

- **Stores secrets, keys, and certificates** — single pane for all sensitive material
- **RBAC access model** — use built-in roles: Key Vault Secrets Officer (manage secrets), Key Vault Secrets User (read secrets), Key Vault Crypto Officer (manage keys), Key Vault Certificates Officer (manage certs)
- **Do not use legacy access policies** — RBAC is the current model, provides finer-grained control and consistency with Azure RBAC
- **Soft delete enabled** — mandatory since February 2025. 90-day retention of deleted secrets.
- **Purge protection for production** — prevents permanent deletion during soft-delete retention period. Enable for all production vaults.
- **Network restrictions** — private endpoint for production Key Vaults. No public access.
- **Diagnostics enabled** — all access logged to Log Analytics. Alert on unusual access patterns.

## Access Patterns

How different Azure services should access Key Vault:

- **App Service / Functions** — Key Vault references in app settings: `@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)`. Managed identity grants access. No secrets in code.
- **AKS** — Secrets Store CSI Driver mounts Key Vault secrets as volumes or environment variables in pods. Workload identity for authentication.
- **VMs** — Azure SDK with managed identity. `DefaultAzureCredential` in code retrieves tokens automatically.
- **Azure Pipelines** — Azure Key Vault task to fetch secrets at pipeline runtime. Secrets not stored in pipeline variables.
- **Terraform / Bicep** — Reference Key Vault secrets in deployment. `getSecret()` in Bicep, `azurerm_key_vault_secret` data source in Terraform.

## Secret Rotation

- **Automated rotation for supported types** — Storage account keys (Event Grid trigger + Azure Function), SQL passwords, Cosmos DB keys. Configure rotation policy in Key Vault.
- **Near-zero-downtime manual rotation pattern:**
  1. Add new secret version to Key Vault
  2. Update all consumers to use new version (or use latest version URI)
  3. Verify all consumers working with new secret
  4. Disable old secret version
  5. After grace period, delete old version
- **Rotation frequency** — 90 days for high-sensitivity secrets, 180 days for standard. Automate wherever possible.
- **Expiry notifications** — configure Key Vault to send Event Grid events before secret expiry

## Detection

Finding secrets where they shouldn't be:

- **GitHub Advanced Security** — secret scanning in repositories, push protection blocks commits containing secrets
- **Azure DevOps Advanced Security** — credential scanning in Azure Repos
- **Pre-commit hooks** — use `detect-secrets` or similar tools to catch secrets before commit
- **Key Vault audit logs** — monitor who accesses secrets, when, and from where. Alert on direct secret reads (should be managed identity, not humans)
- **Defender for Key Vault** — alerts on unusual access patterns, bulk retrieval, access from suspicious IPs

## Anti-Patterns (Always a Finding)

- Secrets in `appsettings.json` or `web.config` committed to source control
- Connection strings stored as pipeline variables (not linked to Key Vault)
- Shared Key Vault across all environments (dev, staging, prod should have separate vaults)
- No rotation policy — secrets never expire and never rotate
- Key Vault using legacy access policies instead of RBAC
- Application code constructing connection strings from individual secrets instead of using managed identity
- Secrets passed as command-line arguments (visible in process listings)
- Logging secret values in application logs or diagnostic output
