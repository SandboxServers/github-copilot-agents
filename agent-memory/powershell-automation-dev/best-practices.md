# Best Practices

PowerShell best practices for Azure automation, covering function design, error handling, logging, idempotency, and module development.

## Advanced Function Template

Every automation function should use this structure as a baseline.

```powershell
function Verb-Noun {
    [CmdletBinding(SupportsShouldProcess, ConfirmImpact = 'Medium')]
    [OutputType([PSCustomObject])]
    param(
        [Parameter(Mandatory, ValueFromPipeline, ValueFromPipelineByPropertyName)]
        [ValidateNotNullOrEmpty()]
        [string[]]$Name,

        [Parameter()]
        [ValidateSet('Option1', 'Option2')]
        [string]$Mode = 'Option1'
    )

    begin   { Write-Verbose "Starting $($MyInvocation.MyCommand)" }
    process {
        foreach ($item in $Name) {
            if ($PSCmdlet.ShouldProcess($item, 'Perform action')) {
                # implementation
            }
        }
    }
    end     { Write-Verbose "Completed $($MyInvocation.MyCommand)" }
}
```

Key elements: `CmdletBinding`, `OutputType`, parameter validation, `ShouldProcess`, pipeline support, `begin`/`process`/`end` blocks.

## Structured Error Handling

Understand the difference between error types and handle each appropriately.

```powershell
$ErrorActionPreference = 'Stop'   # Escalate non-terminating errors

try {
    $resource = Get-AzResource -Name $Name -ErrorAction Stop
} catch [Microsoft.Azure.Commands.ResourceManager.Cmdlets.SdkModels.PSResourceNotFoundException] {
    Write-Warning "Resource '$Name' not found — creating..."
    $resource = New-AzResource @params
} catch {
    $PSCmdlet.ThrowTerminatingError($_)   # Re-throw unknown errors
}
```

- Use `-ErrorAction Stop` on individual commands for granular control.
- Catch specific exception types before the generic `catch` block.
- Use `$PSCmdlet.ThrowTerminatingError()` in advanced functions (not `throw`).

## Logging Strategy

Use PowerShell's built-in streams consistently.

| Stream | Cmdlet | Purpose |
|--------|--------|---------|
| Success (1) | `Write-Output` | Function return data — capturable |
| Verbose (4) | `Write-Verbose` | Step-by-step diagnostic detail |
| Information (6) | `Write-Information` | Operational messages, structured logging |
| Warning (3) | `Write-Warning` | Recoverable issues |
| Error (2) | `Write-Error` | Non-terminating failures |

For audit trails, start a transcript:

```powershell
$transcriptPath = Join-Path $env:TEMP "automation-$(Get-Date -Format 'yyyyMMdd-HHmmss').log"
Start-Transcript -Path $transcriptPath -Append
try { ... } finally { Stop-Transcript }
```

## Idempotent Scripts

Automation scripts must be safe to re-run. Check state before modifying.

```powershell
function Set-TagOnResource {
    [CmdletBinding(SupportsShouldProcess)]
    param([string]$ResourceId, [hashtable]$Tags)

    $resource = Get-AzResource -ResourceId $ResourceId
    $needsUpdate = $false
    foreach ($key in $Tags.Keys) {
        if ($resource.Tags[$key] -ne $Tags[$key]) { $needsUpdate = $true; break }
    }

    if ($needsUpdate -and $PSCmdlet.ShouldProcess($ResourceId, 'Update tags')) {
        Set-AzResource -ResourceId $ResourceId -Tag $Tags -Force
    } else {
        Write-Verbose "Tags already current on $ResourceId"
    }
}
```

## Comment-Based Help

Every exported function needs discoverable help.

```powershell
function Get-Widget {
<#
.SYNOPSIS
    Retrieves widgets from the inventory service.
.DESCRIPTION
    Queries the widget API with optional filtering by status and owner.
.PARAMETER Name
    Widget name. Accepts wildcards and pipeline input.
.EXAMPLE
    Get-Widget -Name 'prod-*' | Export-Csv widgets.csv
    Exports all production widgets to CSV.
.NOTES
    Requires Az.Accounts module and Contributor role.
#>
    [CmdletBinding()]
    param(...)
}
```

## Credential Handling

Never store or pass plain-text credentials. Prefer managed identity; fall back to certificate or SecretManagement.

```powershell
# Priority 1: Managed Identity (Azure Automation, Functions, AKS)
Connect-AzAccount -Identity

# Priority 2: Certificate-based auth
Connect-AzAccount -ServicePrincipal -CertificateThumbprint $thumb -TenantId $tenant -ApplicationId $appId

# Priority 3: SecretManagement vault
$secret = Get-Secret -Name 'ApiKey' -Vault 'AzKeyVault' -AsPlainText
```

## Cross-Platform Patterns

Write scripts that work on Windows and Linux.

```powershell
# BAD — Windows-only backslash paths
$path = "$env:USERPROFILE\Documents\report.csv"

# GOOD — platform-agnostic
$path = Join-Path -Path ([Environment]::GetFolderPath('MyDocuments')) -ChildPath 'report.csv'

# Test platform when needed
if ($IsLinux) { ... } elseif ($IsWindows) { ... }
```

Avoid: registry access, COM objects, Windows-specific environment variables, `\r\n` assumptions.

## Pester Testing Structure

```powershell
Describe 'Get-Widget' {
    BeforeAll {
        . $PSScriptRoot/../src/Get-Widget.ps1
    }

    Context 'when widget exists' {
        BeforeAll {
            Mock Invoke-RestMethod { @{ Name = 'TestWidget'; Status = 'Active' } }
        }

        It 'returns widget with correct name' {
            $result = Get-Widget -Name 'TestWidget'
            $result.Name | Should -Be 'TestWidget'
        }

        It 'calls API exactly once' {
            Should -Invoke Invoke-RestMethod -Times 1 -Exactly
        }
    }
}
```

Mock all external dependencies. Test behavior, not implementation. Use `TestDrive:\` for file system tests.

## Module Manifest Best Practices

```powershell
# Key fields in .psd1
@{
    RootModule        = 'MyModule.psm1'
    ModuleVersion     = '1.2.0'                    # Semantic versioning
    FunctionsToExport = @('Get-Widget', 'Set-Widget')  # Explicit, never '*'
    CmdletsToExport   = @()
    AliasesToExport   = @()
    RequiredModules   = @(
        @{ ModuleName = 'Az.Accounts'; ModuleVersion = '3.0.0' }
    )
    PrivateData = @{ PSData = @{ Prerelease = 'beta1' } }
}
```

Always list exports explicitly — wildcard exports break autoloading performance.

## Performance

- Avoid repeated API calls — cache results in variables or hashtable lookups.
- Use `[System.Collections.Generic.List[object]]` instead of `+=` array growth.
- Batch operations: prefer `Get-AzResource -Tag @{env='prod'}` over per-item queries.
- Pipeline streaming over collect-then-process for large datasets.

> **Sources:** [Chapter 9 - Functions](https://learn.microsoft.com/powershell/scripting/learn/ps101/09-functions), [about_Functions_Advanced](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_functions_advanced), [about_Functions_CmdletBindingAttribute](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_functions_cmdletbindingattribute), [PowerShell scripting performance considerations](https://learn.microsoft.com/powershell/scripting/dev-cross-plat/performance/script-authoring-considerations)
