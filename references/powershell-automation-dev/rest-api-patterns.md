# REST API Patterns

Patterns for calling REST APIs from PowerShell using `Invoke-RestMethod` and `Invoke-WebRequest`, with emphasis on Azure/Graph API scenarios.

## Basic Request Pattern

```powershell
$params = @{
    Uri         = 'https://management.azure.com/subscriptions?api-version=2022-12-01'
    Method      = 'Get'
    Headers     = @{ Authorization = "Bearer $accessToken" }
    ContentType = 'application/json'
}
$response = Invoke-RestMethod @params
```

Always use splatting for readability. Set `-ContentType` explicitly — the default is `application/x-www-form-urlencoded`.

## Authentication Patterns

```powershell
# Azure managed identity token (Functions, Automation, AKS)
$tokenResponse = Invoke-RestMethod -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/' `
    -Headers @{ Metadata = 'true' }
$accessToken = $tokenResponse.access_token

# Az module token (interactive or service principal session)
$token = (Get-AzAccessToken -ResourceUrl 'https://graph.microsoft.com').Token

# OAuth2 client credentials flow
$body = @{
    grant_type    = 'client_credentials'
    client_id     = $clientId
    client_secret = $clientSecret
    scope         = 'https://graph.microsoft.com/.default'
}
$tokenResponse = Invoke-RestMethod -Method Post -Uri "https://login.microsoftonline.com/$tenantId/oauth2/v2.0/token" -Body $body
```

Never hardcode tokens. Prefer managed identity → certificate → SecretManagement vault.

## Pagination

Most Azure and Graph APIs return paged results. Always follow continuation links.

```powershell
function Get-AllPages {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)][string]$Uri,
        [Parameter(Mandatory)][hashtable]$Headers
    )

    $results = [System.Collections.Generic.List[object]]::new()
    $nextLink = $Uri

    while ($nextLink) {
        $response = Invoke-RestMethod -Uri $nextLink -Headers $Headers
        if ($response.value) {
            $results.AddRange($response.value)
        }
        $nextLink = $response.'@odata.nextLink' ?? $response.nextLink
    }

    return $results
}
```

For `Invoke-RestMethod`, PS 7+ supports `-FollowRelLink` and `-MaximumFollowRelLink` for RFC 5988 link headers.

## Retry Logic With Exponential Backoff

Handle transient failures and throttling (HTTP 429, 503).

```powershell
function Invoke-RestMethodWithRetry {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)][hashtable]$Params,
        [int]$MaxRetries = 5,
        [int]$BaseDelaySeconds = 2
    )

    for ($attempt = 1; $attempt -le $MaxRetries; $attempt++) {
        try {
            return Invoke-RestMethod @Params -ErrorAction Stop
        } catch {
            $statusCode = $_.Exception.Response.StatusCode.value__
            if ($statusCode -in @(429, 503, 502, 504) -and $attempt -lt $MaxRetries) {
                $retryAfter = $_.Exception.Response.Headers['Retry-After']
                $delay = if ($retryAfter) { [int]$retryAfter }
                         else { $BaseDelaySeconds * [Math]::Pow(2, $attempt - 1) }
                Write-Warning "HTTP $statusCode — retry $attempt/$MaxRetries in ${delay}s"
                Start-Sleep -Seconds $delay
            } else {
                throw
            }
        }
    }
}
```

PS 7.4+ has built-in `-MaximumRetryCount` and `-RetryIntervalSec` on `Invoke-RestMethod`, but they only retry on 304+ status codes and don't honor `Retry-After` headers.

## Error Handling

`Invoke-RestMethod` treats HTTP 4xx/5xx as terminating errors when using `-ErrorAction Stop`.

```powershell
try {
    $result = Invoke-RestMethod @params -ErrorAction Stop
} catch {
    $statusCode = $_.Exception.Response.StatusCode.value__
    $errorBody  = $_.ErrorDetails.Message | ConvertFrom-Json -ErrorAction SilentlyContinue

    switch ($statusCode) {
        401 { throw "Authentication failed — check token expiry" }
        403 { throw "Insufficient permissions: $($errorBody.error.message)" }
        404 { Write-Warning "Resource not found"; return $null }
        429 { Write-Warning "Throttled — implement retry" }
        default { throw "HTTP $statusCode`: $_" }
    }
}
```

Use `-StatusCodeVariable` (PS 7+) to inspect status without exceptions:

```powershell
$result = Invoke-RestMethod @params -StatusCodeVariable 'statusCode' -SkipHttpErrorCheck
if ($statusCode -ne 200) { Write-Warning "Unexpected status: $statusCode" }
```

## Batch Operations

Azure and Graph APIs support batch requests to reduce round trips.

```powershell
# Microsoft Graph batch (up to 20 requests)
$batch = @{
    requests = @(
        @{ id = '1'; method = 'GET'; url = '/users/user1@contoso.com' }
        @{ id = '2'; method = 'GET'; url = '/users/user2@contoso.com' }
    )
} | ConvertTo-Json -Depth 5

$response = Invoke-RestMethod -Uri 'https://graph.microsoft.com/v1.0/$batch' `
    -Method Post -Body $batch -ContentType 'application/json' -Headers $headers
```

## Request Body Best Practices

```powershell
# Use hashtables + ConvertTo-Json, not string concatenation
$body = @{
    displayName = $name
    properties  = @{
        sku = 'Standard'
        tags = @{ environment = 'production' }
    }
} | ConvertTo-Json -Depth 10   # Always set -Depth for nested objects (default is 2)
```

The default `-Depth 2` silently truncates nested objects — always specify `-Depth 10` or higher.

> **Sources:** [Invoke-RestMethod](https://learn.microsoft.com/powershell/module/microsoft.powershell.utility/invoke-restmethod), [Graph API error handling with Invoke-RestMethod](https://learn.microsoft.com/troubleshoot/entra/entra-id/app-integration/graph-api-error-handling-invoke-restmethod), [Best practices for RESTful web API design](https://learn.microsoft.com/azure/architecture/best-practices/api-design)
