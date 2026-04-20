# Azure Functions with PowerShell

Patterns and practices for writing PowerShell-based Azure Functions, including triggers, bindings, and function app configuration.

## Project Structure

```
FunctionApp/
├── host.json                 # Runtime configuration
├── requirements.psd1         # Module dependencies
├── profile.ps1               # Cold-start initialization
├── HttpTrigger/
│   ├── function.json         # Trigger + binding definitions
│   └── run.ps1               # Function code
├── TimerTrigger/
│   ├── function.json
│   └── run.ps1
└── Modules/                  # Custom shared modules
    └── MyHelpers/
```

## Trigger Patterns

### HTTP Trigger

```json
// function.json
{
    "bindings": [
        { "authLevel": "function", "type": "httpTrigger", "direction": "in",
          "name": "Request", "methods": ["get", "post"] },
        { "type": "http", "direction": "out", "name": "Response" }
    ]
}
```

```powershell
# run.ps1
param($Request, $TriggerMetadata)

$name = $Request.Query.Name ?? $Request.Body.Name ?? 'World'

Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = [System.Net.HttpStatusCode]::OK
    Body       = "Hello, $name"
})
```

### Timer Trigger (Scheduled Tasks)

```json
{
    "bindings": [
        { "name": "Timer", "type": "timerTrigger", "direction": "in",
          "schedule": "0 */5 * * * *" }
    ]
}
```

```powershell
param($Timer)
if ($Timer.IsPastDue) { Write-Warning "Timer is running late!" }
Write-Information "Cleanup triggered at $(Get-Date -Format 'o')" -InformationAction Continue
# perform scheduled work
```

### Queue Trigger

```json
{
    "bindings": [
        { "name": "QueueItem", "type": "queueTrigger", "direction": "in",
          "queueName": "work-items", "connection": "AzureWebJobsStorage" },
        { "name": "OutputQueue", "type": "queue", "direction": "out",
          "queueName": "results", "connection": "AzureWebJobsStorage" }
    ]
}
```

```powershell
param([string]$QueueItem, $TriggerMetadata)
$item = $QueueItem | ConvertFrom-Json
# process...
Push-OutputBinding -Name OutputQueue -Value ($result | ConvertTo-Json -Compress)
```

## Module Management

### requirements.psd1

```powershell
@{
    'Az.Accounts' = '3.*'
    'Az.Resources' = '7.*'
    'Az.KeyVault' = '6.*'
}
```

Functions runtime downloads these on first cold start. Pin major versions; allow minor updates.

### profile.ps1 (Cold-Start Init)

```powershell
# Authenticate with managed identity
if ($env:MSI_SECRET) {
    Connect-AzAccount -Identity | Out-Null
    Write-Information "Connected via managed identity" -InformationAction Continue
}
```

This runs once per worker instance startup. Keep it minimal — it blocks all executions until complete.

## Error Handling in Functions

```powershell
param($Request, $TriggerMetadata)

try {
    $ErrorActionPreference = 'Stop'
    $result = Get-AzResource -Name $Request.Query.Name
    Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
        StatusCode = 200
        Body       = ($result | ConvertTo-Json -Depth 10)
    })
} catch {
    Write-Error "Function failed: $_"
    Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
        StatusCode = 500
        Body       = @{ error = $_.Exception.Message } | ConvertTo-Json
    })
}
```

Unhandled exceptions mark the invocation as failed in Application Insights.

## Performance Considerations

- **Cold start:** PS Functions have ~10-20s cold starts. Use Premium plan or keep-alive pings for latency-sensitive workloads.
- **Managed dependencies:** Slow first load. Consider bundling critical modules in a `Modules/` folder instead of `requirements.psd1`.
- **Concurrency:** PowerShell workers are single-threaded per instance. Scale out is handled by the Functions host adding instances.
- **Output binding batching:** Call `Push-OutputBinding` once with a collection rather than in a loop.

## Environment Variables and Configuration

```powershell
# App Settings are environment variables
$apiKey = $env:MY_API_KEY
$connStr = $env:AzureWebJobsStorage

# Key Vault references (no code changes needed)
# In App Settings: @Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/MySecret)
$secret = $env:MY_SECRET   # Resolved by the platform
```

## Durable Functions (Orchestration)

PowerShell supports Durable Functions for long-running workflows.

```powershell
# Orchestrator function
param($Context)
$output = @()
$output += Invoke-DurableActivity -FunctionName 'Step1' -Input 'data1'
$output += Invoke-DurableActivity -FunctionName 'Step2' -Input $output[0]
return $output
```

Constraints: orchestrator code must be deterministic — no `Get-Date`, `Get-Random`, or direct I/O. Use activity functions for non-deterministic work.

## Deployment

```powershell
# Deploy via Azure Functions Core Tools
func azure functionapp publish $functionAppName --powershell

# Or via Az CLI
az functionapp deployment source config-zip -g $rg -n $functionAppName --src $zipPath
```

> **Sources:** [Azure Functions PowerShell developer guide](https://learn.microsoft.com/azure/azure-functions/functions-reference-powershell), [Azure Functions scenarios](https://learn.microsoft.com/azure/azure-functions/functions-scenarios), [Durable Functions overview](https://learn.microsoft.com/azure/durable-task/common/what-is-durable-task)
