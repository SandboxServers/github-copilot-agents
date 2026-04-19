## Microsoft Module Ecosystem

### Microsoft Graph PowerShell SDK
- Replacement for AzureAD and MSOnline modules (both deprecated)
- Two module flavors: `Microsoft.Graph` (full) and `Microsoft.Graph.Beta` (beta APIs)
- Cross-platform — PS 7 and PS 5.1
- Authentication patterns:
  - **Managed Identity**: `Connect-MgGraph -Identity` (system-assigned) or `Connect-MgGraph -Identity -ClientId <UAMI-ClientId>` (user-assigned)
  - **Certificate**: `Connect-MgGraph -ClientId $appId -TenantId $tenant -CertificateThumbprint $thumb`
  - **Client secret** (non-interactive): `Connect-MgGraph -ClientSecretCredential $cred -TenantId $tenant`
  - **Interactive**: `Connect-MgGraph -Scopes "User.Read.All","Group.Read.All"`
- Scopes/permissions: request minimum needed, use `Find-MgGraphCommand` to discover required permissions
- Pagination: most `Get-Mg*` cmdlets handle pagination automatically — use `-All` for all results
- **Gotcha**: default page size is small (100), always use `-All` or `-PageSize` for bulk operations
- **Gotcha**: `-Filter` uses OData syntax, not PowerShell comparison operators
- **Gotcha**: some beta cmdlets change signature between releases — pin module versions

### Az Modules
- `Az.Accounts`, `Az.Resources`, `Az.Compute`, `Az.Network`, `Az.Storage`, etc.
- Authentication: `Connect-AzAccount` (interactive), `Connect-AzAccount -Identity` (managed identity), `Connect-AzAccount -ServicePrincipal -CertificateThumbprint`
- Context management: `Set-AzContext`, `Get-AzContext`, `Select-AzSubscription`
- **Gotcha**: `Az` and `AzureRM` cannot coexist in the same session
- **Gotcha**: some Az cmdlets don't support `-ErrorAction Stop` correctly — always test
- **Gotcha**: `Get-AzResourceGroup` returns nothing (not error) when resource group doesn't exist — check `$null`

### Exchange Online Management
- Module: `ExchangeOnlineManagement`
- Requires PS execution policy `RemoteSigned`
- Supports PS 7.x with .NET 6/7/8 depending on module version
- Auth: `Connect-ExchangeOnline -CertificateThumbprint $thumb -AppId $appId -Organization contoso.onmicrosoft.com`
- **Gotcha**: REST-based cmdlets (V3) replace older RPS-based cmdlets — use `Get-EXO*` cmdlets for better performance
- **Gotcha**: connection limit per tenant — connection pooling not built in
- **Gotcha**: some cmdlets have different output properties in REST mode vs RPS mode

### Microsoft Teams Module
- Module: `MicrosoftTeams`
- Auth: `Connect-MicrosoftTeams -CertificateThumbprint $thumb -ApplicationId $appId -TenantId $tenant`
- Covers: team management, channels, policies, phone number assignment
- **Gotcha**: Teams module doesn't cover all Teams admin scenarios — some need Graph API or Teams Admin Center API directly

### PnP PowerShell
- Module: `PnP.PowerShell` — community-maintained, .NET 8.0 based
- 700+ cmdlets for SharePoint Online, Teams, Entra, Planner
- Auth: `Connect-PnPOnline -Url https://contoso.sharepoint.com -ClientId $id -CertificatePath cert.pfx`
- **No Microsoft SLA** — community support only
- **Gotcha**: PnP module versions tightly coupled to SharePoint API changes — pin versions
- **Gotcha**: `Connect-PnPOnline` needs explicit `-Url` per site — no tenant-wide connection

### When to Skip the Module and Hit the REST API
- Module doesn't support the operation (e.g., advanced Graph beta endpoints)
- Module has throttling issues you can handle better directly
- Need to batch requests (Graph `$batch` endpoint)
- Module output doesn't include all properties (use `Invoke-MgGraphRequest` or `Invoke-RestMethod`)
- Module version is buggy and you can't upgrade

## Graph API Throttling & Retry

### Throttling Signals
- HTTP 429 Too Many Requests + `Retry-After` header
- HTTP 503 Service Unavailable (backoff immediately)
- SharePoint uses resource unit-based throttling — `RateLimit-Limit`, `RateLimit-Remaining` headers

### Retry Pattern
```powershell
function Invoke-GraphWithRetry {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Uri,
        [int]$MaxRetries = 5
    )
    $attempt = 0
    while ($attempt -lt $MaxRetries) {
        try {
            return Invoke-MgGraphRequest -Uri $Uri -Method GET -ErrorAction Stop
        }
        catch {
            $response = $_.Exception.Response
            if ($response.StatusCode -eq 429 -or $response.StatusCode -eq 503) {
                $retryAfter = $response.Headers['Retry-After']
                $waitSeconds = if ($retryAfter) { [int]$retryAfter } else { [math]::Pow(2, $attempt) }
                Write-Warning "Throttled. Waiting $waitSeconds seconds (attempt $($attempt + 1)/$MaxRetries)"
                Start-Sleep -Seconds $waitSeconds
                $attempt++
            }
            else { throw }
        }
    }
    throw "Failed after $MaxRetries attempts"
}
```

### Best Practices
- Honor `Retry-After` header — always
- Use exponential backoff as fallback
- Reduce concurrent requests — don't parallelize Graph calls aggressively
- Use delta queries (delta with token = 1 resource unit in SharePoint)
- Batch requests: `$batch` endpoint for up to 20 requests in one call
- Use `$select` to return only needed properties — reduces response size and processing cost
