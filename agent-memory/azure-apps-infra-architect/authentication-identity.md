# Authentication & Identity

## Managed Identity (Always Preferred)

```csharp
// DefaultAzureCredential tries managed identity first, then falls back
var credential = new DefaultAzureCredential();
var client = new SecretClient(new Uri(kvUri), credential);
```

System-assigned: tied to resource lifecycle. One identity per resource. Deleted when resource deleted.
User-assigned: independent lifecycle. Can be shared across resources. Create once, assign many.

### Services That Support Managed Identity

Key Vault, Storage, SQL Database, Service Bus, Event Hubs, Cosmos DB, Azure OpenAI, App Configuration, Redis Cache (Entra auth), Container Registry

### RBAC Roles (Least Privilege)

| Service | Read Role | Write Role |
|---------|-----------|------------|
| Key Vault | Key Vault Secrets User | Key Vault Secrets Officer |
| Storage Blobs | Storage Blob Data Reader | Storage Blob Data Contributor |
| SQL Database | db_datareader (DB level) | db_datawriter (DB level) |
| Service Bus | Azure Service Bus Data Receiver | Azure Service Bus Data Sender |
| Cosmos DB | Cosmos DB Built-in Data Reader | Cosmos DB Built-in Data Contributor |

## App Service Easy Auth

Built-in authentication middleware. Zero code changes. Supports Entra ID, Google, Facebook, GitHub, Apple. Configure in portal or Bicep. Returns X-MS-CLIENT-PRINCIPAL header to your app.

## Key Vault Integration

App Service Key Vault references: `@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/)` in app settings. Requires managed identity with Key Vault Secrets User role.

## Service-to-Service Auth

Managed identity for Azure-to-Azure. Workload identity federation for external (GitHub Actions, AKS pods, other clouds). No secrets stored anywhere.
