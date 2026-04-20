# Anti-Patterns

Common PowerShell anti-patterns, especially in Azure automation scripts. Recognizing these patterns prevents bugs, security issues, and maintenance headaches.

## Write-Host Instead of Proper Output Streams

`Write-Host` writes directly to the console, bypassing the pipeline. Output cannot be captured, redirected, or tested.

```powershell
# BAD — lost to the console
Write-Host "Found $count users"

# GOOD — use the correct stream for the intent
Write-Output $result              # Success pipeline (capturable)
Write-Verbose "Processing $user"  # Diagnostic detail (-Verbose to see)
Write-Information "Status: OK"    # Informational (6> redirect, -InformationAction)
```

`Write-Host` is acceptable only for interactive formatting (colors, progress bars) that should never be captured.

## Missing CmdletBinding and Param Blocks

Without `[CmdletBinding()]`, functions lack `-Verbose`, `-ErrorAction`, `-WhatIf`, and pipeline support.

```powershell
# BAD
function Get-Data { param($Name) ... }

# GOOD
function Get-Data {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Name
    )
    ...
}
```

## Aliases in Scripts

Aliases (`ls`, `?`, `%`, `where`, `foreach`, `cd`, `cat`) are for interactive use only. They reduce readability, break cross-platform, and violate PSScriptAnalyzer rules.

```powershell
# BAD
ls | ? { $_.Length -gt 1MB } | % { $_.Name }

# GOOD
Get-ChildItem | Where-Object { $_.Length -gt 1MB } | ForEach-Object { $_.Name }
```

## String Concatenation

Repeated `+` concatenation is slower, harder to read, and error-prone with types.

```powershell
# BAD
$msg = "User " + $name + " has " + $count + " items"

# GOOD — string interpolation
$msg = "User $name has $count items"

# GOOD — format operator (locale-safe, reusable)
$msg = 'User {0} has {1:N0} items' -f $name, $count
```

## No Error Handling

Scripts that ignore errors silently produce incorrect results in automation.

```powershell
# BAD — failures silently continue
Get-AzResource -Name $name
Remove-AzResource -Id $id

# GOOD — explicit error strategy
$ErrorActionPreference = 'Stop'
try {
    $resource = Get-AzResource -Name $name
    Remove-AzResource -Id $resource.Id -Force
} catch {
    Write-Error "Failed to remove resource '$name': $_"
}
```

## Global Variables Instead of Parameters

Global/script-scope variables create hidden coupling, make testing impossible, and break parallelism.

```powershell
# BAD
$Global:TenantId = 'abc-123'
function Connect-Tenant { Connect-AzAccount -TenantId $Global:TenantId }

# GOOD
function Connect-Tenant {
    [CmdletBinding()]
    param([Parameter(Mandatory)][string]$TenantId)
    Connect-AzAccount -TenantId $TenantId
}
```

## Ignoring Pipeline Input

Functions that only accept parameter input miss PowerShell's composability.

```powershell
# GOOD — support both parameter and pipeline
param(
    [Parameter(Mandatory, ValueFromPipeline, ValueFromPipelineByPropertyName)]
    [string[]]$ComputerName
)
process {
    foreach ($computer in $ComputerName) { ... }
}
```

## Non-Approved Verbs

Using verbs not in `Get-Verb` output triggers PSScriptAnalyzer warnings, confuses discoverability, and breaks `Get-Command` verb filtering. Use `Get-`, `Set-`, `New-`, `Remove-`, `Invoke-`, `Start-`, `Stop-`, `Test-`, etc.

## Hardcoded Credentials

Never embed secrets, passwords, or tokens in source code. Use `SecretManagement`, managed identity, or Key Vault references.

```powershell
# BAD — credential in source
$password = 'P@ssw0rd!'

# GOOD — SecretManagement vault
$secret = Get-Secret -Name 'MyApiKey' -Vault 'AzKeyVault'
```

## Using Invoke-Expression

`Invoke-Expression` (`iex`) is a code injection vector. Almost every use case has a safer alternative.

```powershell
# BAD — injection risk
Invoke-Expression "Get-Process -Name $userInput"

# GOOD — use the call operator or splatting
& Get-Process -Name $userInput
```

## Missing ShouldProcess Support

Destructive functions without `-WhatIf`/`-Confirm` are dangerous in automation pipelines.

```powershell
function Remove-Widget {
    [CmdletBinding(SupportsShouldProcess, ConfirmImpact = 'High')]
    param([Parameter(Mandatory)][string]$Name)
    if ($PSCmdlet.ShouldProcess($Name, 'Remove widget')) {
        # perform deletion
    }
}
```

## Blocking Without Progress

Long-running operations without feedback leave operators guessing if the script is hung.

```powershell
# Use Write-Progress for batch operations
$items | ForEach-Object -Begin { $i = 0 } -Process {
    Write-Progress -Activity 'Processing' -Status $_.Name -PercentComplete (($i++ / $total) * 100)
    ...
}
```

## COM Objects When Cmdlets Exist

COM (e.g., `New-Object -ComObject Excel.Application`) is fragile, Windows-only, and leaks memory. Use `ImportExcel`, `Export-Csv`, or Graph SDK equivalents instead.

> **Sources:** [PowerShell scripting performance considerations](https://learn.microsoft.com/powershell/scripting/dev-cross-plat/performance/script-authoring-considerations), [Advisory Development Guidelines](https://learn.microsoft.com/powershell/scripting/developer/cmdlet/advisory-development-guidelines), [Entra PowerShell best practices](https://learn.microsoft.com/powershell/entra-powershell/entra-powershell-best-practices)
