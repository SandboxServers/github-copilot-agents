# Performance Optimization

## Benchmarking
Always measure before optimizing. Gut feelings about performance are usually wrong.
```powershell
# Basic timing
$elapsed = Measure-Command {
    $result = Get-ChildItem -Path C:\ -Recurse -File -ErrorAction SilentlyContinue
}
Write-Output "Elapsed: $($elapsed.TotalSeconds) seconds"

# Compare two approaches
$approach1 = Measure-Command { <# code A #> }
$approach2 = Measure-Command { <# code B #> }
Write-Output "A: $($approach1.TotalMilliseconds)ms vs B: $($approach2.TotalMilliseconds)ms"

# Trace-Command for deep debugging of specific subsystem behavior
Trace-Command -Name ParameterBinding -Expression { Get-Process -Name pwsh } -PSHost
```

## Collection Operations (The #1 Performance Trap)
```powershell
# BAD: Array += in a loop — O(n²) because it copies the entire array each iteration
$results = @()
foreach ($item in $largeCollection) {
    $results += $item.Name    # NEVER do this in loops
}

# GOOD: Generic List — O(1) per add
$results = [System.Collections.Generic.List[string]]::new()
foreach ($item in $largeCollection) {
    $results.Add($item.Name)
}

# BEST: Array subexpression (let PowerShell handle it)
$results = @(foreach ($item in $largeCollection) {
    $item.Name    # Output to pipeline, PS collects into array
})

# Also good: Pipeline (slightly slower than foreach statement, but readable)
$results = $largeCollection | ForEach-Object { $_.Name }
```

## String Building
```powershell
# BAD: String concatenation in loops (same problem as array +=)
$output = ''
foreach ($line in $lines) { $output += "$line`n" }

# GOOD: StringBuilder for large string assembly
$sb = [System.Text.StringBuilder]::new()
foreach ($line in $lines) { [void]$sb.AppendLine($line) }
$output = $sb.ToString()

# GOOD: -join operator (clean and fast for simple cases)
$output = $lines -join "`n"

# GOOD: Join-String (PS 7.0+ pipeline-friendly)
$output = $lines | Join-String -Separator "`n"
```

## Hashtable Lookups vs Where-Object
```powershell
# BAD: Linear search — O(n) per lookup, O(n*m) for matching two collections
$user = $users | Where-Object { $_.Id -eq $targetId }

# GOOD: Build a hashtable index — O(1) per lookup, O(n+m) total
$userIndex = @{}
foreach ($user in $users) {
    $userIndex[$user.Id] = $user
}
# Now lookups are instant
$matchedUser = $userIndex[$targetId]

# Group-Object -AsHashtable shortcut (PS 7.0+)
$userIndex = $users | Group-Object -Property Id -AsHashtable
```

## Output Suppression
```powershell
# When you don't need the return value (e.g., List.Add() returns index)
[void]$list.Add($item)         # Fastest — compile-time cast
$null = $list.Add($item)       # Second fastest — assignment to $null
$list.Add($item) | Out-Null    # Slowest — pipeline overhead. Avoid in loops
$list.Add($item) > $null       # Same as $null = but less idiomatic
```

## Object Creation
```powershell
# GOOD: [PSCustomObject] accelerator (fastest for PS objects)
$obj = [PSCustomObject]@{
    Name   = $user.Name
    Email  = $user.Email
    Status = 'Active'
}

# AVOID: New-Object (3-5x slower than accelerator)
$obj = New-Object PSObject -Property @{ Name = $user.Name }  # Don't use this

# For high-volume: .NET types are fastest
$obj = [PSCustomObject]@{ Name = $n; Value = $v }  # Good enough for most cases
```

## File I/O
```powershell
# GOOD for moderate files
$content = Get-Content -Path $path -Raw        # Read entire file as one string
$lines = Get-Content -Path $path               # Read as string array (line by line)

# BETTER for large files (100MB+): .NET StreamReader
$reader = [System.IO.StreamReader]::new($path)
try {
    while ($null -ne ($line = $reader.ReadLine())) {
        # Process line by line — constant memory usage
    }
}
finally {
    $reader.Dispose()
}

# Writing large files
$writer = [System.IO.StreamWriter]::new($outputPath)
try {
    foreach ($item in $largeCollection) {
        $writer.WriteLine($item)
    }
}
finally {
    $writer.Dispose()
}

# Quick reads: [System.IO.File] static methods
$content = [System.IO.File]::ReadAllText($path)
$lines = [System.IO.File]::ReadAllLines($path)
```

## Filtering: Pipeline vs Statement
```powershell
# Pipeline Where-Object (readable, slower for large sets)
$active = $users | Where-Object { $_.Enabled -eq $true }

# .Where() method (faster, no pipeline overhead)
$active = $users.Where({ $_.Enabled -eq $true })

# foreach statement with if (fastest for simple filtering)
$active = @(foreach ($user in $users) {
    if ($user.Enabled) { $user }
})

# Provider-level filtering (fastest of all — filters at source)
Get-ChildItem -Path C:\ -Filter '*.log'          # Filesystem filter (WildcardPattern)
Get-ADUser -Filter "Enabled -eq '$true'"          # AD server-side filter
Get-MgUser -Filter "accountEnabled eq true"       # Graph OData filter
Search-AzGraph -Query "where type == 'microsoft.compute/virtualmachines'"
```

## Caching Patterns
```powershell
# Module-scoped cache with TTL
$script:cache = @{}
$script:cacheTTL = [TimeSpan]::FromMinutes(5)

function Get-CachedData {
    param([string]$Key, [scriptblock]$Factory)

    $entry = $script:cache[$Key]
    if ($entry -and ((Get-Date) - $entry.Timestamp) -lt $script:cacheTTL) {
        return $entry.Value
    }

    $value = & $Factory
    $script:cache[$Key] = @{ Value = $value; Timestamp = Get-Date }
    return $value
}

# Usage
$users = Get-CachedData -Key 'AllUsers' -Factory { Get-MgUser -All }
```

## When to Use .NET Directly
| Scenario | PowerShell | .NET Alternative |
|---|---|---|
| Large file I/O | `Get-Content` | `[System.IO.File]::ReadAllLines()` |
| HTTP requests | `Invoke-RestMethod` | `[System.Net.Http.HttpClient]` |
| String assembly | `+= ` or `-join` | `[StringBuilder]` |
| Regex (repeated) | `-match` | `[regex]::new()` (compile once) |
| Collection building | `@() +=` | `[Generic.List[T]]` |

Decision: Use .NET when processing >10,000 items or when profiling shows cmdlet overhead dominates. For normal automation (<1000 items), cmdlets are fine and more readable.

## Memory Management
- Pipeline processing is memory-efficient — one object at a time
- `foreach` statement loads entire collection into memory first
- For huge datasets: stream with pipeline or StreamReader
- Avoid `Select-Object *` — explicitly select needed properties
- Large variables: set to `$null` when done (helps GC in long-running scripts)
- `[GC]::Collect()` — almost never needed. If you think you need it, you probably have a design problem.

## Parallel Processing Decision Tree
1. Less than ~50 items? → Sequential. Runspace overhead isn't worth it.
2. CPU-bound work? → Limited benefit. Consider compiled code (.NET).
3. I/O-bound with 50+ items? → `ForEach-Object -Parallel -ThrottleLimit N`
4. Need fine-grained control? → Runspace pool pattern.
5. Long-running background? → `Start-Job` or `Start-ThreadJob`.
