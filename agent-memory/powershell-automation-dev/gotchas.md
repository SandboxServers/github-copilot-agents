# Gotchas

PowerShell behaviors that surprise even experienced developers. Understanding these prevents subtle bugs in automation scripts.

## Single-Element Array Unrolling

PowerShell unrolls single-element arrays to scalars. A command returning one item gives you the item, not an array.

```powershell
$results = Get-ChildItem *.log   # Might return 0, 1, or many files
$results.GetType()               # String (1 file) vs Object[] (2+ files)

# FIX — force array context
[array]$results = Get-ChildItem *.log
# or
$results = @(Get-ChildItem *.log)
```

This is the single most common source of "method not found" errors in automation.

## Return Behavior — Everything Is Output

Every uncaptured expression in a function is emitted to the pipeline, not just `return` statements.

```powershell
function Get-Data {
    "Starting..."              # This is OUTPUT, not a log message
    $connection = Open-DB       # If Open-DB returns something, it's also output
    $data = Get-Records
    return $data               # Now output includes "Starting...", connection, AND $data
}

# FIX — capture or suppress unwanted output
function Get-Data {
    Write-Verbose "Starting..."
    $null = Open-DB
    $data = Get-Records
    return $data
}
```

Suppress with `$null =`, `[void]`, or `| Out-Null`. Never use `Write-Host` to "fix" this.

## Comparison Operators With Arrays

When the left operand is an array, `-eq` returns **matching elements**, not a boolean.

```powershell
$data = @('red', 'green', 'blue')
$data -eq 'green'    # Returns 'green' (the string), not $true
$data -ne 'green'    # Returns @('red', 'blue'), not $true

# Gotcha: this is always "true" because -ne returns non-empty array
if ($data -ne 'green') { "This always runs!" }

# FIX — use -contains or -in for boolean tests
if ($data -contains 'green') { ... }
if ('green' -in $data) { ... }
```

## Case-Insensitive by Default

All string comparison operators are case-insensitive. Use `c` prefix variants for case-sensitive matching.

```powershell
'Hello' -eq 'hello'    # True (case-insensitive)
'Hello' -ceq 'hello'   # False (case-sensitive)
'ABC' -match '[a-z]+'  # True (case-insensitive regex)
'ABC' -cmatch '[a-z]+' # False (case-sensitive regex)
```

## $null Comparison Order

When comparing with `$null`, always put `$null` on the left side.

```powershell
# BAD — if $array is an array, this filters $null elements instead of testing
if ($array -eq $null) { ... }

# GOOD — scalar comparison, returns boolean
if ($null -eq $array) { ... }
```

## Scope Inheritance

Child scopes (functions, script blocks) can **read** parent variables but cannot **write** them. Assignments create new local copies.

```powershell
$counter = 0
function Increment { $counter++ }   # Creates LOCAL $counter; parent unchanged
Increment
$counter   # Still 0

# FIX — explicit scope modifier (use sparingly)
function Increment { $script:counter++ }
```

Prefer parameter passing and return values over scope manipulation.

## ForEach-Object -Parallel Scoping

In `-Parallel` blocks, local variables are not accessible. Use `$using:` to bring them in.

```powershell
$threshold = 100
1..10 | ForEach-Object -Parallel {
    # $threshold is $null here!
    if ($_ -gt $using:threshold) { "Over threshold" }
}
```

Imported variables are **copies** — mutations are not reflected back. Complex objects are serialized/deserialized.

## Automatic Module Autoloading

PowerShell 3+ auto-imports modules when you call their commands. This masks missing `Import-Module` statements during development that will fail in constrained environments (Azure Functions, containers, CI runners).

```powershell
# Works in dev due to autoloading, fails in Azure Automation
Get-AzResource -Name 'myvm'

# Explicit is safer
#Requires -Modules Az.Resources
Import-Module Az.Resources -ErrorAction Stop
```

## Set-StrictMode Differences

`Set-StrictMode -Version Latest` changes behavior across PowerShell versions. Version 3+ treats accessing non-existent properties as errors.

```powershell
Set-StrictMode -Version Latest
$obj = [PSCustomObject]@{ Name = 'Test' }
$obj.DoesNotExist   # Error in strict mode, $null otherwise
```

Use `-Version 3.0` for consistent cross-version behavior. Be aware that strict mode does NOT apply in child modules.

## PSCustomObject vs Hashtable

They look similar but behave differently in pipelines and formatting.

```powershell
$hash = @{ Name = 'A'; Value = 1 }          # Hashtable — no pipeline member enumeration
$obj  = [PSCustomObject]@{ Name = 'A'; Value = 1 }  # PSCustomObject — proper pipeline citizen

$hash | Select-Object Name    # Returns nothing useful
$obj  | Select-Object Name    # Returns @{ Name = 'A' }
```

Always use `[PSCustomObject]` for pipeline output. Use hashtables for lookups and splatting.

## Double-Hop Authentication

PowerShell remoting cannot forward credentials to a second hop (e.g., remote server → file share). The second hop sees anonymous access.

```powershell
# Fails: credential not forwarded to \\fileserver
Invoke-Command -ComputerName Server1 -ScriptBlock {
    Get-ChildItem \\fileserver\share
}

# Mitigations: CredSSP (least secure), Kerberos delegation, managed identity, or copy data first
```

In Azure, prefer managed identity for cross-resource access instead of remoting chains.

## Module Version Conflicts

Multiple module versions can load simultaneously, causing unpredictable behavior.

```powershell
# Check loaded versions
Get-Module Az.Accounts -ListAvailable

# Force specific version
Import-Module Az.Accounts -RequiredVersion 3.0.0 -Force

# In manifests, pin versions
RequiredModules = @(@{ ModuleName = 'Az.Accounts'; RequiredVersion = '3.0.0' })
```

> **Sources:** [Everything you wanted to know about arrays](https://learn.microsoft.com/powershell/scripting/learn/deep-dives/everything-about-arrays), [about_Return](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_return), [about_Arrays](https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_arrays), [PowerShell Language Specification §3.5](https://learn.microsoft.com/powershell/scripting/lang-spec/chapter-03)
