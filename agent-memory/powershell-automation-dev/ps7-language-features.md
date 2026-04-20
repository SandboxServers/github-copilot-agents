# PowerShell 7 Language Features

## Architecture: .NET Framework (5.1) vs .NET 8+ (7.x)
- PS 5.1: Windows-only, .NET Framework 4.x, ships with Windows
- PS 7.x: Cross-platform, .NET 8 (7.4) / .NET 9 (7.5), separate install
- Side-by-side: Both can coexist. `powershell.exe` = 5.1, `pwsh` = 7.x
- API availability: Most .NET APIs available, some Windows-specific ones are not (WPF, WinForms require `-UseWindowsPowerShell`)

## New Operators

### Ternary Operator (7.0+)
```powershell
$status = $isEnabled ? 'Active' : 'Disabled'
# Use when the condition and branches are short and obvious
# For complex logic, stick with if/else — readability wins
```

### Null-Coalescing (7.0+)
```powershell
# ?? returns left if not null, otherwise right
$displayName = $user.DisplayName ?? 'Unknown User'

# ??= assigns only if currently null
$config.RetryCount ??= 3
```

### Null-Conditional Member Access (7.1+)
```powershell
# ?. returns null instead of throwing if left side is null
$length = ${object}?.Property?.Length
# Note: requires ${} syntax for variable names with ?
```

### Pipeline Chain Operators (7.0+)
```powershell
# && runs second command only if first succeeds
dotnet build && dotnet test

# || runs second command only if first fails
Get-Process -Name 'myapp' -ErrorAction SilentlyContinue || Write-Warning 'App not running'

# These check $? (last command success), NOT exit codes for native commands (before 7.4)
```

## ForEach-Object -Parallel (7.0+)
Deep-dive into the parallel execution model:

```powershell
# Basic parallel processing with throttle
$results = 1..100 | ForEach-Object -Parallel {
    # Each iteration runs in its own runspace
    $item = $_
    # External variables require $using: prefix
    $baseUrl = $using:apiBaseUrl
    
    Invoke-RestMethod -Uri "$baseUrl/items/$item"
} -ThrottleLimit 10
```

### When to use (and when not to)
- USE: I/O-bound work (API calls, file operations, network requests)
- USE: Processing hundreds+ of independent items
- AVOID: CPU-bound work (PowerShell's GIL means limited CPU parallelism)
- AVOID: Small collections — runspace overhead exceeds time savings
- AVOID: When items depend on each other or share mutable state

### Thread-Safe Patterns
```powershell
# Thread-safe collection for collecting results
$errors = [System.Collections.Concurrent.ConcurrentBag[string]]::new()

$items | ForEach-Object -Parallel {
    $errorBag = $using:errors
    try {
        # Do work
    }
    catch {
        $errorBag.Add("Failed for $_ : $($_.Exception.Message)")
    }
} -ThrottleLimit 5

# Check for errors after parallel execution
if ($errors.Count -gt 0) {
    Write-Warning "Encountered $($errors.Count) errors"
    $errors | ForEach-Object { Write-Warning $_ }
}
```

### Runspace Variables
```powershell
# $using: is required for ALL external variables
$tenantId = 'abc-123'
$token = Get-AzAccessToken

$users | ForEach-Object -Parallel {
    $tid = $using:tenantId    # Copies value into runspace
    $tok = $using:token       # Copies the token object
    # Module imports happen per-runspace
    Import-Module Microsoft.Graph.Users
    Connect-MgGraph -AccessToken $tok.Token
}
```

## Clean Block (7.3+)
```powershell
function Get-SomeData {
    [CmdletBinding()]
    param([string]$Source)
    
    begin {
        $connection = Open-Connection -Source $Source
    }
    process {
        # Process pipeline input
        Get-Data -Connection $connection
    }
    clean {
        # Runs ALWAYS — even on Ctrl+C / pipeline stops
        # Unlike finally (which doesn't run on pipeline termination)
        if ($connection) {
            Close-Connection -Connection $connection
            Write-Verbose "Connection cleaned up"
        }
    }
}
```

## $PSStyle (7.2+)
```powershell
# ANSI escape sequence formatting
$PSStyle.Formatting.Warning = $PSStyle.Foreground.Yellow
$PSStyle.Progress.View = 'Classic'  # or 'Minimal'

# Use in output
Write-Host "$($PSStyle.Foreground.Green)Success$($PSStyle.Reset)"

# $PSStyle.OutputRendering controls when ANSI codes are emitted
$PSStyle.OutputRendering = 'PlainText'  # Strip ANSI for log files
```

## Enhanced Cmdlets

### Get-Error (7.0+)
```powershell
# Detailed error inspection — replaces digging through $Error[0]
Get-Error             # Most recent error with full detail
Get-Error -Newest 3   # Last 3 errors
# Shows: Exception, InnerException, StackTrace, ScriptStackTrace, ErrorRecord details
```

### Web Cmdlets Improvements
```powershell
# Follow pagination links automatically
$allResults = Invoke-RestMethod -Uri $uri -FollowRelLink -MaximumFollowRelLink 10

# Capture response headers
$response = Invoke-RestMethod -Uri $uri -ResponseHeadersVariable 'headers'
$headers['x-ratelimit-remaining']

# Skip certificate validation (development only!)
Invoke-RestMethod -Uri $uri -SkipCertificateCheck
```

### ConvertFrom-Json Improvements
```powershell
# Return hashtable instead of PSCustomObject (faster, easier to manipulate)
$data = Get-Content config.json | ConvertFrom-Json -AsHashtable

# Specify depth for complex objects
$deep = $json | ConvertFrom-Json -Depth 10
```

### Other New Cmdlets
```powershell
# Join-String: pipeline-friendly string concatenation
'one', 'two', 'three' | Join-String -Separator ', '           # "one, two, three"
Get-Process | Join-String -Property Name -Separator ', '       # "proc1, proc2, ..."

# Test-Json with schema validation
$isValid = $jsonString | Test-Json -SchemaFile './schema.json'

# Group-Object -AsHashtable for fast lookups
$grouped = Get-Process | Group-Object -Property Name -AsHashtable
$grouped['pwsh']  # Direct lookup, no Where-Object needed
```

## PS 7.4+ Specific
- `$PSNativeCommandUseErrorActionPreference = $true` — native command failures respect ErrorAction
- Sealed/abstract class members
- `ValidateNotNullOrWhiteSpace` attribute
- Tab completion improvements

## Removed/Changed Modules
Modules that don't exist in PS7 (Windows PowerShell only):
- `Microsoft.PowerShell.LocalAccounts`
- `PSScheduledJob`
- `ISE` module
- Most `DCOM`-based cmdlets

### Windows Compatibility Layer
```powershell
# Import a Windows PowerShell-only module into PS7
Import-Module ActiveDirectory -UseWindowsPowerShell
# Creates an implicit remoting session to Windows PowerShell
# Performance cost — each call crosses process boundary via WinRM
```

## SSH Remoting (7.0+)
```powershell
# Remoting over SSH instead of WinRM
$session = New-PSSession -HostName server01 -UserName admin -SSHTransport
Invoke-Command -Session $session -ScriptBlock { Get-Process }

# Requires: sshd running on target, Subsystem powershell in sshd_config
# Works cross-platform (Windows ↔ Linux ↔ macOS)
```

## Experimental Features
```powershell
# List available experimental features
Get-ExperimentalFeature

# Enable a feature (persists in powershell.config.json)
Enable-ExperimentalFeature -Name PSCommandNotFoundSuggestion

# Disable
Disable-ExperimentalFeature -Name PSCommandNotFoundSuggestion
# Restart pwsh for changes to take effect
```

## Migration Checklist (5.1 → 7.x)
Things that break or change:
- `WMI` cmdlets → use `CIM` cmdlets instead
- `-ComputerName` on many cmdlets → use `Invoke-Command -Session` or CIM sessions
- `$Host.UI.RawUI.WindowTitle` works differently cross-platform
- Default encoding changed to UTF-8 (no BOM) — was ASCII in 5.1
- Some type accelerators differ (removed some .NET Framework specifics)
- PowerShell Gallery modules may need updating for compatibility
