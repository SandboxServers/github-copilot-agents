# Execution Environments

## Environment Comparison Matrix

| Environment | Runtime Limit | Scaling | Auth Options | Module Management | Cost Model |
|---|---|---|---|---|---|
| Azure Automation | 3 hours (cloud) / unlimited (hybrid) | Job queue | Managed identity, Run As (deprecated), cert | Runtime environment (managed/custom) | Per-minute |
| Azure Functions | 5-60 min (depends on plan) | Event-driven autoscale | Managed identity, app settings | managed dependencies or bundle | Consumption/Premium/Dedicated |
| GitHub Actions | 6 hours (default) | Parallel jobs | OIDC (WIF), secrets | Install-Module per run | Free tier + per-minute |
| Azure Pipelines | 60 min (default, configurable) | Agent pool | Service connection (WIF/cert), variable groups | Install-Module per run | Parallel job minutes |
| Container (ACI/Container Apps) | Unlimited | Manual/KEDA | Managed identity, env vars | Baked into image | Per-second |
| Local Scheduled Task | Unlimited | None | Current user, gMSA | Pre-installed or Install-Module | Infrastructure cost |
| Hybrid Runbook Worker | Unlimited | Manual | Azure Automation assets + local resources | Local + Automation modules | Azure Automation pricing |

## Azure Automation

### Runtime Environments (Replaces old module management)
```powershell
# Azure Automation now uses Runtime Environments
# System-managed: PowerShell 5.1 or 7.2, pre-installed Az modules
# Custom: Your own module set via requirements.psd1
```

### Key Gotchas
- 3-hour execution limit for cloud runbooks (use Hybrid Worker for longer)
- No interactive prompts — `Read-Host`, `Get-Credential` will fail
- Output truncation: max 1MB per stream (Verbose, Warning, Output, Error)
- `Write-Progress` is silently ignored
- Sandbox: no access to local filesystem paths, no registry
- Managed identity is the only supported auth going forward (Run As deprecated)

### Hybrid Runbook Worker
```powershell
# Hybrid Worker runs on your own VM — gets local resource access
# Setup: Install the Hybrid Worker extension on an Azure VM
# Or: Install the agent on an on-prem server connected via Azure Arc

# Benefits:
# - No 3-hour time limit
# - Access local files, network shares, on-prem databases
# - Install any module (not limited to Automation sandbox)
# - Run as a specific gMSA or local account

# Auth: Can use Automation managed identity OR local credentials
Connect-AzAccount -Identity  # Works on Hybrid Worker too
```

### Structured Logging in Automation
```powershell
# Automation captures Write-Output, Write-Verbose, Write-Warning, Write-Error as separate streams
# View in: Automation Account → Jobs → [Job] → All Logs

# For Log Analytics integration, enable diagnostic settings on the Automation Account
# Sends to: AzureDiagnostics table with Category = 'JobStreams'
```

## Azure Functions (PowerShell)

### Profile and Requirements
```powershell
# profile.ps1 — runs once per worker cold start
# Use for: connecting to Azure, setting up shared state
Connect-AzAccount -Identity | Out-Null

# requirements.psd1 — managed dependencies
@{
    'Az.Accounts' = '2.*'
    'Az.Storage'  = '6.*'
}
# Azure Functions automatically installs and caches these
```

### Trigger Patterns
```powershell
# HTTP Trigger
param($Request, $TriggerMetadata)
$name = $Request.Query.Name ?? $Request.Body.Name ?? 'World'
Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = [HttpStatusCode]::OK
    Body       = "Hello, $name"
})

# Timer Trigger (cron schedule)
param($Timer)
# $Timer.IsPastDue indicates if the function missed its schedule
if ($Timer.IsPastDue) { Write-Warning "Timer is past due!" }
# Do scheduled work...

# Queue Trigger
param([string]$QueueItem, $TriggerMetadata)
$message = $QueueItem | ConvertFrom-Json
Write-Output "Processing: $($message.Id)"

# Event Grid Trigger
param($eventGridEvent, $TriggerMetadata)
Write-Output "Event type: $($eventGridEvent.eventType)"
Write-Output "Subject: $($eventGridEvent.subject)"
```

### Cold Start Mitigation
- Premium plan: pre-warmed instances (no cold start)
- Keep `profile.ps1` minimal — heavy init delays first invocation
- Use `requirements.psd1` managed dependencies instead of `Install-Module` at runtime
- Module bundling: include modules in function app for Consumption plan
```powershell
# Bundled modules in function app
# Set in host.json: "managedDependency": { "enabled": false }
# Then include modules in /Modules folder of function app
```

## GitHub Actions

### PowerShell in Workflows
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest    # pwsh is pre-installed on all GitHub runners
    permissions:
      id-token: write          # Required for OIDC
      contents: read
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Azure Login (OIDC)
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      
      - name: Run PowerShell Script
        shell: pwsh
        run: |
          # Az context is already set from azure/login
          $rg = Get-AzResourceGroup -Name 'my-rg'
          Write-Output "Found: $($rg.ResourceGroupName)"
      
      - name: Run Script File
        shell: pwsh
        run: ./scripts/Deploy-Infrastructure.ps1 -Environment 'production'
```

### Module Caching
```yaml
      - name: Cache PowerShell Modules
        uses: actions/cache@v4
        with:
          path: ~/.local/share/powershell/Modules
          key: ps-modules-${{ hashFiles('**/requirements.psd1') }}
      
      - name: Install Modules
        shell: pwsh
        run: |
          if (-not (Get-Module -ListAvailable Az.Accounts)) {
            Install-Module Az.Accounts, Az.Resources -Force -Scope CurrentUser
          }
```

## Azure Pipelines

### PowerShell Tasks
```yaml
# Inline PowerShell (PS 7)
- task: PowerShell@2
  displayName: 'Run Deployment Script'
  inputs:
    targetType: inline
    pwsh: true                # true = pwsh (PS 7), false = powershell.exe (5.1)
    script: |
      Write-Output "Running on: $($PSVersionTable.PSVersion)"
      ./scripts/Deploy.ps1 -Environment '$(Environment)'

# Script file
- task: PowerShell@2
  displayName: 'Run Script File'
  inputs:
    filePath: './scripts/Deploy.ps1'
    arguments: '-Environment "$(Environment)" -Verbose'
    pwsh: true

# AzurePowerShell task (handles auth automatically via service connection)
- task: AzurePowerShell@5
  displayName: 'Azure PowerShell with Service Connection'
  inputs:
    azureSubscription: 'my-service-connection'   # WIF or cert-based
    ScriptType: InlineScript
    Inline: |
      # Az context is pre-configured — just use Az cmdlets
      Get-AzResourceGroup | Format-Table Name, Location
    azurePowerShellVersion: LatestVersion
    pwsh: true
```

### Variable Groups and Secrets
```powershell
# Access pipeline variables
$environment = $env:ENVIRONMENT        # Mapped from $(Environment)
$secret = $env:MY_SECRET               # Secret variables are masked in logs

# Output variables for downstream jobs
Write-Host "##vso[task.setvariable variable=DeployedVersion;isOutput=true]1.2.3"
```

## Container-Based Execution (ACI / Container Apps)

### Dockerfile for PowerShell Automation
```dockerfile
FROM mcr.microsoft.com/powershell:7.4-alpine-3.20

# Install modules at build time (cached in image layer)
SHELL ["pwsh", "-Command"]
RUN Install-Module Az.Accounts, Az.Resources, Microsoft.Graph.Users -Force -Scope AllUsers

# Copy scripts
COPY scripts/ /app/scripts/
WORKDIR /app

# Entry point
ENTRYPOINT ["pwsh", "-File", "/app/scripts/Main.ps1"]
```

### When to Use Containers
- Long-running jobs exceeding Azure Automation limits
- Complex dependency chains (specific module versions, native tools)
- Jobs that need both PowerShell and other tools (az CLI, terraform, etc.)
- Consistent environment across dev/test/prod

## Idempotency Pattern (All Environments)
```powershell
function Set-ResourceState {
    [CmdletBinding(SupportsShouldProcess)]
    param(
        [Parameter(Mandatory)] [string]$ResourceGroupName,
        [Parameter(Mandatory)] [string]$Location
    )
    
    # Check current state
    $existing = Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue
    
    if ($existing) {
        Write-Verbose "Resource group '$ResourceGroupName' already exists — no action needed"
        return [PSCustomObject]@{
            Name   = $ResourceGroupName
            Status = 'AlreadyExists'
            Action = 'None'
        }
    }
    
    if ($PSCmdlet.ShouldProcess($ResourceGroupName, 'Create Resource Group')) {
        $rg = New-AzResourceGroup -Name $ResourceGroupName -Location $Location
        Write-Verbose "Created resource group '$ResourceGroupName'"
        return [PSCustomObject]@{
            Name   = $ResourceGroupName
            Status = 'Created'
            Action = 'Created'
        }
    }
}
```

## Application Insights Integration
```powershell
# Send custom events to Application Insights from PowerShell
$instrumentationKey = $env:APPINSIGHTS_INSTRUMENTATIONKEY
$telemetryBody = @{
    name = 'Microsoft.ApplicationInsights.Event'
    time = (Get-Date).ToUniversalTime().ToString('o')
    iKey = $instrumentationKey
    data = @{
        baseType = 'EventData'
        baseData = @{
            name       = 'ScriptCompleted'
            properties = @{
                ScriptName = 'Deploy-Infrastructure'
                Duration   = $elapsed.TotalSeconds.ToString()
                Status     = 'Success'
            }
        }
    }
} | ConvertTo-Json -Depth 10

Invoke-RestMethod -Uri 'https://dc.services.visualstudio.com/v2/track' -Method POST -Body $telemetryBody -ContentType 'application/json'
```

## Environment Variable Handling
```powershell
# Cross-platform environment variable access
$value = $env:MY_VARIABLE                        # Works everywhere

# Setting (current process only)
$env:MY_VARIABLE = 'value'

# Platform-specific persistent setting
if ($IsWindows) {
    [Environment]::SetEnvironmentVariable('MY_VAR', 'value', 'User')
}
if ($IsLinux -or $IsMacOS) {
    # Append to shell profile
    Add-Content -Path '~/.bashrc' -Value "export MY_VAR='value'"
}
```
