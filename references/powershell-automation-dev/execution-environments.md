## Execution Environments

### Azure Automation
- **Runtime**: PS 7.4 (or 5.1 legacy) via Runtime Environments
- **Modules**: install from PSGallery via Automation Account → Modules; or bundle with runbook
- **Auth**: managed identity (system or user-assigned), certificate-based stored as Automation Certificate
- **Scheduling**: built-in schedules, webhook triggers, event-driven via Event Grid
- **Gotcha**: modules must be compatible with the runtime — modules that use WinForms, WPF, or COM won't work
- **Gotcha**: `Get-AutomationVariable`, `Get-AutomationCertificate`, `Get-AutomationConnection` — special cmdlets, not Az cmdlets
- **Gotcha**: 3-hour max execution (fair share) for cloud jobs — use Hybrid Runbook Worker for longer
- **Gotcha**: no interactive prompts — any `Read-Host` or `$host.UI` calls will fail
- **Gotcha**: limited output size — large Write-Output calls may be truncated
- **Gotcha**: PowerShell modules installed in portal only work with the selected runtime version

### Azure Functions (PowerShell)
- **Runtime**: PS 7.4 on Functions v4
- **Managed dependencies**: `requirements.psd1` declares modules, auto-installed at cold start
- **Bundled modules**: `Modules/` folder in function app for version-pinned modules
- **Gotcha**: Flex Consumption plan doesn't support managed dependencies — bundle modules instead
- **Gotcha**: cold start penalty — managed dependencies download on first execution after deployment
- **Gotcha**: `host.json` must have `"managedDependency": { "enabled": true }` for managed dependencies
- **Best for**: event-driven, HTTP-triggered automation that needs fast scaling

### GitHub Actions Runners
- **Runtime**: whatever PowerShell is on the runner image (typically PS 7.x on ubuntu-latest)
- **Auth**: OIDC federation with `azure/login` action → `Connect-AzAccount` with federated token
- **Modules**: install inline with `Install-Module -Force -Scope CurrentUser`
- **Best for**: CI/CD-triggered automation, deployment scripts, cross-platform workflows

### Local Scheduled Tasks
- **Windows**: Task Scheduler → `pwsh.exe -File script.ps1`
- **Linux**: cron → `pwsh -File /path/to/script.ps1`
- **Auth**: service account, certificate, or stored credential
- **Best for**: on-prem operations, hybrid scenarios, quick one-off scheduling
- **Gotcha**: no built-in retry, logging, or alerting — build it yourself

## Production Script Patterns

### Idempotency
```powershell
# Check-then-act pattern
$rg = Get-AzResourceGroup -Name $rgName -ErrorAction SilentlyContinue
if (-not $rg) {
    $rg = New-AzResourceGroup -Name $rgName -Location $location
    Write-Verbose "Created resource group: $rgName"
} else {
    Write-Verbose "Resource group already exists: $rgName"
}
```

### Structured Logging
```powershell
function Write-Log {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)][string]$Message,
        [ValidateSet('Info','Warning','Error')][string]$Level = 'Info'
    )
    $timestamp = Get-Date -Format 'yyyy-MM-ddTHH:mm:ss.fffZ' -AsUTC
    $entry = [pscustomobject]@{
        Timestamp = $timestamp
        Level     = $Level
        Message   = $Message
    }
    switch ($Level) {
        'Info'    { Write-Information "[$timestamp] $Message" -InformationAction Continue }
        'Warning' { Write-Warning "[$timestamp] $Message" }
        'Error'   { Write-Error "[$timestamp] $Message" }
    }
    $entry  # Also output to pipeline for log collection
}
```
