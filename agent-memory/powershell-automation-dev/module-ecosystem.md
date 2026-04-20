# Module Ecosystem

## Microsoft Graph PowerShell SDK

### Architecture
- V2 SDK uses sub-modules: `Microsoft.Graph.Users`, `Microsoft.Graph.Groups`, `Microsoft.Graph.Applications`, etc.
- Install only what you need — full SDK is 100+ modules: `Install-Module Microsoft.Graph.Users`
- Beta endpoint: `Microsoft.Graph.Beta.Users` (separate modules)
- `Microsoft.Graph.Authentication` is always required (auto-installed as dependency)

### Authentication
```powershell
# Interactive (development)
Connect-MgGraph -Scopes 'User.Read.All', 'Group.Read.All'

# Managed identity
Connect-MgGraph -Identity

# Certificate
Connect-MgGraph -ClientId $appId -TenantId $tenantId -Certificate $x509Cert

# Client secret (avoid if possible — prefer cert or managed identity)
$clientSecret = Get-Secret -Name 'GraphSecret' -AsPlainText
$credential = [PSCredential]::new($appId, (ConvertTo-SecureString $clientSecret -AsPlainText -Force))
# Note: Client secret auth requires MSAL.PS or direct token acquisition
# Graph SDK doesn't support client secret directly — use certificate instead
```

### Pagination
```powershell
# Cmdlets handle pagination automatically with -All
$allUsers = Get-MgUser -All -Property 'Id,DisplayName,SignInActivity'

# Manual pagination for custom requests
$uri = 'https://graph.microsoft.com/v1.0/users?$top=100'
$allResults = @()
do {
    $response = Invoke-MgGraphRequest -Method GET -Uri $uri
    $allResults += $response.value
    $uri = $response.'@odata.nextLink'
} while ($uri)
```

### Invoke-MgGraphRequest (Escape Hatch)
```powershell
# For endpoints not covered by cmdlets or when you need raw control
$response = Invoke-MgGraphRequest -Method GET -Uri 'https://graph.microsoft.com/v1.0/me'

# POST with body
$body = @{
    displayName = 'New Group'
    mailNickname = 'newgroup'
    groupTypes = @('Unified')
    mailEnabled = $true
    securityEnabled = $false
} | ConvertTo-Json

Invoke-MgGraphRequest -Method POST -Uri 'https://graph.microsoft.com/v1.0/groups' -Body $body
```

### Batch Requests
```powershell
# Batch up to 20 requests in a single HTTP call
$batchBody = @{
    requests = @(
        @{ id = '1'; method = 'GET'; url = '/users/user1@contoso.com' }
        @{ id = '2'; method = 'GET'; url = '/users/user2@contoso.com' }
        @{ id = '3'; method = 'GET'; url = '/users/user3@contoso.com' }
    )
} | ConvertTo-Json -Depth 5

$batchResponse = Invoke-MgGraphRequest -Method POST -Uri 'https://graph.microsoft.com/v1.0/$batch' -Body $batchBody
# Process individual responses
$batchResponse.responses | ForEach-Object {
    if ($_.status -eq 200) { $_.body }
    else { Write-Warning "Request $($_.id) failed: $($_.status)" }
}
```

### Delta Queries (Incremental Sync)
```powershell
# Initial sync — gets all users + deltaLink
$deltaUri = 'https://graph.microsoft.com/v1.0/users/delta?$select=displayName,mail'
$allChanges = @()
do {
    $response = Invoke-MgGraphRequest -Method GET -Uri $deltaUri
    $allChanges += $response.value
    $deltaUri = $response.'@odata.nextLink'
} while ($response.'@odata.nextLink')

# Save the deltaLink for next run
$deltaLink = $response.'@odata.deltaLink'
Set-Content -Path './deltaLink.txt' -Value $deltaLink

# Subsequent run — only gets changes since last sync
$deltaUri = Get-Content -Path './deltaLink.txt'
$changes = (Invoke-MgGraphRequest -Method GET -Uri $deltaUri).value
```

### Permission Discovery
```powershell
# Find required permissions for a cmdlet
Find-MgGraphCommand -Command 'Get-MgUser' | Select-Object -ExpandProperty Permissions
```

### Gotchas
- OData filters use single quotes: `-Filter "displayName eq 'John Doe'"`
- `$select` requires `-Property` parameter on cmdlets
- Some properties need `$expand`: `Get-MgUser -UserId $id -ExpandProperty 'Manager'`
- Graph returns max 999 results by default — always use `-All` or paginate

## Az PowerShell Modules

### Authentication
```powershell
# Managed identity (Azure Automation, Functions, VMs)
Connect-AzAccount -Identity

# Service principal with certificate
Connect-AzAccount -ServicePrincipal -CertificateThumbprint $thumb -ApplicationId $appId -TenantId $tenantId

# Context management
Get-AzContext                                    # Current context
Set-AzContext -Subscription 'my-subscription'    # Switch subscription
Get-AzSubscription | Where-Object State -eq 'Enabled'  # List active subs
```

### Cross-Subscription Queries with Resource Graph
```powershell
# Query across ALL subscriptions at once
$query = @"
Resources
| where type == 'microsoft.compute/virtualmachines'
| where properties.provisioningState == 'Succeeded'
| project name, resourceGroup, location, properties.hardwareProfile.vmSize
"@
$results = Search-AzGraph -Query $query -First 1000
```

### ARM Throttling (Different from Graph)
- ARM throttling is per-subscription and per-resource-provider
- Limits: 12,000 reads/hour, 1,200 writes/hour per subscription
- Check headers: `x-ms-ratelimit-remaining-subscription-reads`
- Retry with `Retry-After` header value
```powershell
# Throttling-aware wrapper for ARM operations
function Invoke-AzWithThrottleHandling {
    [CmdletBinding()]
    param([scriptblock]$ScriptBlock, [int]$MaxRetries = 5)
    
    for ($i = 0; $i -lt $MaxRetries; $i++) {
        try {
            return & $ScriptBlock
        }
        catch {
            if ($_.Exception.Response.StatusCode -eq 429) {
                $retryAfter = $_.Exception.Response.Headers['Retry-After'] ?? 30
                Write-Warning "ARM throttled. Waiting $retryAfter seconds..."
                Start-Sleep -Seconds $retryAfter
            }
            else { throw }
        }
    }
}
```

### Common Gotchas
- `Get-AzResource` is slow for large environments — use `Search-AzGraph` instead
- `-ErrorAction Stop` is essential in try blocks — many Az cmdlets emit non-terminating errors
- Context persistence: `Disable-AzContextAutosave` in automation to prevent context leaks

## Exchange Online Management
```powershell
# Connect with managed identity (recommended)
Connect-ExchangeOnline -ManagedIdentity -Organization 'contoso.onmicrosoft.com'

# Connect with certificate
Connect-ExchangeOnline -CertificateThumbprint $thumb -AppId $appId -Organization 'contoso.onmicrosoft.com'

# REST-based cmdlets (v3+) — no more legacy RPS
# All cmdlets now use REST by default. No action needed.
```
- `Disconnect-ExchangeOnline -Confirm:$false` in finally blocks
- Some cmdlets still have RPS equivalents but REST is default now
- Max 3 concurrent connections per account

## Microsoft Teams Module
```powershell
Connect-MicrosoftTeams -Identity  # Managed identity
# Or certificate-based for automation
$teamParams = @{
    CertificateThumbprint = $thumb
    ApplicationId         = $appId
    TenantId             = $tenantId
}
Connect-MicrosoftTeams @teamParams
```
- Limited cmdlet coverage — many Teams operations require Graph API
- Use `Invoke-MgGraphRequest` for Teams endpoints not covered by cmdlets

## PnP PowerShell (Community Module for SharePoint/Teams)
```powershell
# Install
Install-Module PnP.PowerShell

# Connect
Connect-PnPOnline -Url 'https://contoso.sharepoint.com/sites/teamsite' -ManagedIdentity
# Or interactive: Connect-PnPOnline -Url $siteUrl -Interactive

# PnP batch operations for SharePoint
$batch = New-PnPBatch
1..100 | ForEach-Object {
    Add-PnPListItem -List 'Documents' -Values @{ Title = "Item $_" } -Batch $batch
}
Invoke-PnPBatch -Batch $batch
```
- Community-maintained, not Microsoft-official
- Broader SharePoint coverage than Microsoft.Graph modules
- Supports managed identity, certificate, interactive auth

## SharePoint Online Management Shell
```powershell
# Separate from PnP — Microsoft's official module for tenant-level SharePoint admin
Install-Module Microsoft.Online.SharePoint.PowerShell

Connect-SPOService -Url 'https://contoso-admin.sharepoint.com'
# Tenant-level operations: site creation, storage quotas, sharing policies
```

## When to Skip Modules and Use REST Directly
| Use REST When | Use Module When |
|---|---|
| Endpoint not covered by cmdlets | Cmdlet exists and handles pagination |
| Need batch requests ($batch) | Standard CRUD operations |
| Custom headers/throttle handling needed | Auth is simpler with Connect-* |
| Performance-critical bulk operations | Discoverability matters (Get-Command) |

```powershell
# Direct REST with Az token
$token = (Get-AzAccessToken -ResourceUrl 'https://graph.microsoft.com/').Token
$headers = @{ Authorization = "Bearer $token"; 'Content-Type' = 'application/json' }
$response = Invoke-RestMethod -Uri 'https://graph.microsoft.com/v1.0/me' -Headers $headers
```

## Graph Throttling Retry Pattern
```powershell
function Invoke-GraphWithRetry {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)] [hashtable]$RequestParams,
        [int]$MaxRetries = 5
    )
    for ($attempt = 1; $attempt -le $MaxRetries; $attempt++) {
        try {
            return Invoke-MgGraphRequest @RequestParams
        }
        catch {
            $statusCode = $_.Exception.Response.StatusCode.value__
            if ($statusCode -eq 429 -and $attempt -lt $MaxRetries) {
                $retryAfter = [int]($_.Exception.Response.Headers['Retry-After'] ?? 10)
                Write-Warning "Graph 429 — retry $attempt/$MaxRetries after ${retryAfter}s"
                Start-Sleep -Seconds $retryAfter
            }
            elseif ($statusCode -in 502, 503, 504 -and $attempt -lt $MaxRetries) {
                $delay = [math]::Pow(2, $attempt)
                Write-Warning "Graph $statusCode — retry $attempt/$MaxRetries after ${delay}s"
                Start-Sleep -Seconds $delay
            }
            else { throw }
        }
    }
}
```

## Module Version Pinning
```powershell
# In Azure Automation requirements.psd1
@{
    'Microsoft.Graph.Users'      = '2.*'      # Latest 2.x
    'Az.Accounts'                = '2.13.0'   # Exact version
    'Az.Resources'               = '6.*'      # Latest 6.x
}

# In scripts with #Requires
#Requires -Modules @{ ModuleName = 'Az.Accounts'; RequiredVersion = '2.13.0' }
```
