## Authentication Patterns

### Managed Identity (Azure Automation, Functions, VMs)
```powershell
# Az modules
Connect-AzAccount -Identity
# Graph SDK — system-assigned
Connect-MgGraph -Identity
# Graph SDK — user-assigned
Connect-MgGraph -Identity -ClientId '00000000-0000-0000-0000-000000000000'
```
- System-assigned: tied to resource lifecycle, one per resource
- User-assigned: independent lifecycle, can share across resources
- **Best for**: Azure Automation runbooks, Azure Functions, VMs, Container Apps

### Certificate-Based Auth (Service Principals)
```powershell
Connect-AzAccount -ServicePrincipal -TenantId $tenant `
    -ApplicationId $appId -CertificateThumbprint $thumbprint
Connect-MgGraph -ClientId $appId -TenantId $tenant `
    -CertificateThumbprint $thumbprint
Connect-ExchangeOnline -CertificateThumbprint $thumbprint `
    -AppId $appId -Organization contoso.onmicrosoft.com
```
- Preferred over client secrets for production workloads
- Certificates stored in Azure Key Vault or local cert store
- **Gotcha**: cert must be in `Cert:\CurrentUser\My` or `Cert:\LocalMachine\My` for thumbprint auth

### Workload Identity Federation (Pipelines)
- GitHub Actions: OIDC token → Azure, no secrets stored
- Azure Pipelines: workload identity federation service connection
- No credentials to rotate — identity proven by platform token
- **Pattern**: `az login --federated-token` or `Connect-AzAccount` with federated token

### SecretManagement Module (PS 7)
```powershell
Install-Module Microsoft.PowerShell.SecretManagement
Install-Module Microsoft.PowerShell.SecretStore  # local vault
Register-SecretVault -Name 'AzKeyVault' -ModuleName Az.KeyVault -VaultParameters @{ AZKVaultName = 'my-vault' }
$secret = Get-Secret -Name 'ApiKey' -Vault 'AzKeyVault'
```
- Abstraction layer — same cmdlets regardless of vault backend
- Extension vaults: Azure Key Vault, KeePass, LastPass, HashiCorp Vault, CredMan
- Cross-platform local store with SecretStore module
- **Best practice**: use SecretManagement for dev/local, managed identity for production

### Credential Management
```powershell
# In Azure Automation
$credential = Get-AutomationPSCredential -Name 'ServiceAccount'
# In Azure Functions
$secret = Get-AzKeyVaultSecret -VaultName $vaultName -Name $secretName -AsPlainText
# In local dev with SecretManagement
$apiKey = Get-Secret -Name 'ApiKey' -AsPlainText
# NEVER: hardcode credentials, store in source, use ConvertTo-SecureString with plaintext key
```
