## Performance Optimization

### Collection Operations
```powershell
# BAD: array addition — O(n²) for large collections
$results = @()
foreach ($item in $collection) { $results += $item }

# GOOD: Generic List
$results = [System.Collections.Generic.List[object]]::new()
foreach ($item in $collection) { $results.Add($item) }

# GOOD: Pipeline to array expression
$results = @(foreach ($item in $collection) { Process-Item $item })

# BEST for typed data: Generic List[T]
$ids = [System.Collections.Generic.List[int]]::new()
```

### String Building
```powershell
# BAD: string concatenation in loops
$output = ""
foreach ($line in $lines) { $output += "$line`n" }

# GOOD: StringBuilder
$sb = [System.Text.StringBuilder]::new()
foreach ($line in $lines) { [void]$sb.AppendLine($line) }

# GOOD: -join operator
$output = ($lines | ForEach-Object { $_ }) -join "`n"
```

### Output Suppression
```powershell
# FAST: assign to $null
$null = $list.Add($item)
# FAST: cast to [void]
[void]$list.Add($item)
# FAST: redirect to $null
$list.Add($item) > $null
# SLOW (especially in 5.1): pipe to Out-Null
$list.Add($item) | Out-Null
```

### Object Creation
```powershell
# FAST: PSCustomObject accelerator
[pscustomobject]@{ Name = 'test'; Value = 42 }
# FAST: .NET new() static method
[System.IO.FileInfo]::new('path')
# SLOW: New-Object cmdlet
New-Object -TypeName PSObject -Property @{ Name = 'test'; Value = 42 }
```

### General Performance Rules
- PowerShell JIT-compiles loops over 16 iterations (loops < 300 instructions eligible)
- Avoid `Write-Host` in high-throughput code — `[Console]::WriteLine()` is faster but non-portable
- Functions have overhead — for tight loops, consider inlining logic
- Direct .NET method calls are faster than cmdlet equivalents but less idiomatic
- `Where-Object { }` is slower than `.Where({ })` method syntax
- `ForEach-Object { }` is slower than `foreach ($x in $collection) { }` statement
- `Get-Content` is slow for large files — use `[System.IO.File]::ReadLines()` or `[StreamReader]`
- Use `-Filter` on cmdlets (filters at provider level) instead of `Where-Object` (filters in pipeline)
