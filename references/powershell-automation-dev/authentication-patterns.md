# Authentication Patterns

## Auth Hierarchy (Prefer Top to Bottom)
1. Managed Identity (system-assigned preferred)
2. Workload Identity Federation (OIDC — no secrets)
3. Certificate-based (service principal + X.509)
4. SecretManagement module (local development)
5. Interactive (development only, never in automation)

## Managed Identity

### System-Assigned (Preferred)
```powershell
# Azure Automation, Azure Functions, Azure VMs
Connect-AzAccount -Identity

# Verify identity
$context = Get-AzContext
Write-Verbose "Authenticated as: $($context.Account.Id)"
```

### User-Assigned
```powershell
Connect-AzAccount -Identity -AccountId '<client-id-of-user-assigned-mi>'
```

### With Microsoft Graph
```powershell
Connect-MgGraph -Identity
# For specific scopes (not needed with app permissions, but required with delegated)
Connect-MgGraph -Identity -ClientId '<client-id>'
```

## Workload Identity Federation (OIDC)

### GitHub Actions
```yaml
# In workflow YAML - set up OIDC
permissions:
  id-token: write
  contents: read

steps:
  - name: Azure Login
    uses: azure/login@v2
    with:
      client-id: ${{ secrets.AZURE_CLIENT_ID }}
      tenant-id: ${{ secrets.AZURE_TENANT_ID }}
      subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```
```powershell
# PowerShell in GitHub Actions after azure/login
# Az context is already set — just use it
$context = Get-AzContext
Write-Output "Connected to $($context.Subscription.Name)"
```

### Azure Pipelines OIDC
```powershell
# In Azure Pipelines with workload identity federation service connection
# Use AzurePowerShell@5 task — handles OIDC token exchange automatically
# task: AzurePowerShell@5
# inputs:
#   azureSubscription: 'my-wif-service-connection'
#   ScriptType: 'InlineScript'
#   Inline: |
#     Get-AzContext | Select-Object Account, Subscription
```

## Certificate-Based Authentication

### From Local Certificate Store
```powershell
$certThumbprint = 'ABC123...'
$connectParams = @{
    ApplicationId  = '<app-registration-client-id>'
    CertThumbprint = $certThumbprint
    TenantId       = '<tenant-id>'
}
Connect-AzAccount @connectParams
```

### From Azure Key Vault (Retrieve + Connect)
```powershell
# First connect with managed identity to get the cert
Connect-AzAccount -Identity

# Retrieve certificate from Key Vault
$cert = Get-AzKeyVaultCertificate -VaultName 'MyKeyVault' -Name 'MyCert'
$secret = Get-AzKeyVaultSecret -VaultName 'MyKeyVault' -Name $cert.Name -AsPlainText

# Convert to X509
$certBytes = [Convert]::FromBase64String($secret)
$x509 = [System.Security.Cryptography.X509Certificates.X509Certificate2]::new($certBytes)

# Connect to Graph with the certificate
Connect-MgGraph -Certificate $x509 -ClientId '<client-id>' -TenantId '<tenant-id>'
```

## Token Acquisition for REST Calls
```powershell
# Get raw access token for Azure Resource Manager
$token = Get-AzAccessToken -ResourceUrl 'https://management.azure.com/'
$headers = @{ Authorization = "Bearer $($token.Token)" }
Invoke-RestMethod -Uri $uri -Headers $headers

# Get token for Microsoft Graph
$graphToken = Get-AzAccessToken -ResourceUrl 'https://graph.microsoft.com/'

# Get token for custom API
$customToken = Get-AzAccessToken -ResourceUrl 'api://<your-app-id>'
```

## SecretManagement Module (Local Development)
```powershell
# Install the framework + a vault backend
Install-Module Microsoft.PowerShell.SecretManagement
Install-Module Microsoft.PowerShell.SecretStore

# Register a vault
Register-SecretVault -Name 'LocalDev' -ModuleName Microsoft.PowerShell.SecretStore

# Store and retrieve secrets
Set-Secret -Name 'GraphClientSecret' -Vault 'LocalDev' -Secret (Read-Host -AsSecureString)
$secret = Get-Secret -Name 'GraphClientSecret' -Vault 'LocalDev' -AsPlainText

# Az Key Vault as a vault backend
Install-Module Az.KeyVault
Register-SecretVault -Name 'AzKV' -ModuleName Az.KeyVault -VaultParameters @{
    AZKVaultName = 'MyKeyVault'
    SubscriptionId = '<sub-id>'
}
```

## Permission Scoping / Least Privilege
- Start with the minimum permissions needed, expand only when required
- Use `Find-MgGraphCommand -CommandName 'Get-MgUser'` to discover required Graph permissions
- Use `-Scopes` parameter on `Connect-MgGraph` for delegated scenarios
- For app permissions, configure in Azure AD app registration — not at runtime
- Document required permissions in comment-based help for every script

## Auth Anti-Patterns (Never Do This)
- Never store credentials in scripts, environment variables on shared systems, or source control
- Never use `ConvertTo-SecureString -AsPlainText -Force` with a hardcoded password
- Never use client secrets when certificates or managed identity are available
- Never disable certificate validation (`ServerCertificateValidationCallback`)
- Never share service principal credentials between environments (dev/staging/prod get separate SPs)

## Auth Context Management
```powershell
# Switch between subscriptions
Set-AzContext -Subscription '<subscription-name-or-id>'

# Run against multiple subscriptions
$subscriptions = Get-AzSubscription | Where-Object State -eq 'Enabled'
foreach ($sub in $subscriptions) {
    Set-AzContext -Subscription $sub.Id | Out-Null
    # Do work in this subscription context
}

# Disconnect cleanly in finally blocks
try {
    Connect-AzAccount -Identity
    # ... do work ...
}
finally {
    Disconnect-AzAccount -ErrorAction SilentlyContinue | Out-Null
    Disconnect-MgGraph -ErrorAction SilentlyContinue | Out-Null
}
```
