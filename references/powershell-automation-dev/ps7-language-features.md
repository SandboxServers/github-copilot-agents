## PowerShell 7 (Core) vs Windows PowerShell 5.1

### Architecture
- Windows PowerShell 5.1 = .NET Framework 4.5+ — Windows only, ships in-box
- PowerShell 7.x = .NET (Core) — cross-platform (Windows, macOS, Linux), side-by-side install
- Version lineage: 7.0 (.NET Core 3.1 LTS) → 7.2 (.NET 6 LTS) → 7.4 (.NET 8 LTS) → 7.6 (.NET 10 LTS)
- `pwsh.exe` vs `powershell.exe` — different executables, different paths, different `$PSModulePath`
- `$PSEdition` = `Core` (PS7) vs `Desktop` (5.1) — use for edition detection

### PS7 Language Features (Not Available in 5.1)
- **Ternary operator**: `$result = $condition ? 'yes' : 'no'`
- **Null-coalescing**: `$value = $x ?? 'default'`
- **Null-coalescing assignment**: `$x ??= 'fallback'`
- **Null-conditional member access**: `${obj}?.Property`, `${arr}?[0]` — note `${}` required
- **Pipeline chain operators**: `command1 && command2` (run second if first succeeds), `command1 || command2` (run second if first fails)
- **`ForEach-Object -Parallel`**: built-in parallel iteration with `-ThrottleLimit`
- **`clean` block**: runs after `begin`/`process`/`end` regardless of errors (PS 7.3+)
- **Ternary in string interpolation**: `"Status: $($ok ? 'good' : 'bad')"`
- **`$PSStyle`**: ANSI formatting control (PS 7.2+) — `$PSStyle.Formatting.Error`, `.Warning`, `.Verbose`
- **`$PSNativeCommandUseErrorActionPreference`**: non-zero exit codes emit errors (experimental 7.3, stable 7.4)

### ForEach-Object -Parallel Deep Dive
```powershell
$logNames | ForEach-Object -Parallel {
    Get-WinEvent -LogName $_ -MaxEvents 10000
} -ThrottleLimit 5
```
- Each iteration runs in its own runspace (PS 7.0 = new runspace per iteration, PS 7.1+ = runspace pool reused)
- Use `$using:variableName` to pass variables from caller scope into parallel script blocks
- **Thread safety**: referenced objects must be thread-safe — use `[System.Collections.Concurrent.ConcurrentDictionary]`, `[ConcurrentBag]`, etc.
- `$PipelineVariable` common parameter NOT supported in parallel
- Non-terminating errors appear in non-deterministic order
- Terminating errors stop that iteration only — other parallel blocks continue
- **When to use**: I/O-bound work, large dataset processing, calling external APIs. **When NOT to use**: trivial operations (runspace overhead dominates), CPU-bound work that's already fast

### Pipeline Chain Operators (&&, ||)
```powershell
ssh-keygen -t rsa -b 2048 && Get-Content -Raw ~/.ssh/id_rsa.pub | clip
Build-Project || Send-FailureNotification
```
- Based on `$?` and `$LASTEXITCODE` — works with native commands
- Lower precedence than piping (`|`) or redirection (`>`), higher than assignment (`=`) or semicolons (`;`)
- Left-associative — `a && b || c` groups as `(a && b) || c`

### Modules Removed in PS7 (Breaking from 5.1)
- `PSScheduledJob` — no PS7 equivalent, use OS task scheduler
- `PSWorkflow` — no PS7 equivalent, Windows Workflow Foundation not in .NET Core
- `Microsoft.PowerShell.LocalAccounts` — gone on non-Windows
- `ISE` module — use VS Code + PowerShell extension instead
- `Microsoft.PowerShell.ODataUtils`, `Microsoft.PowerShell.Operation.Validation`

### Compatibility with 5.1 Modules
- .NET Standard 2.0 bridge — many Windows PowerShell modules work unmodified in PS7
- `Import-Module -UseWindowsPowerShell` — creates proxy module using local 5.1 process (Windows only)
- Check compatibility: https://aka.ms/PSModuleCompat
- Modules using WMI directly (not CIM) need `-UseWindowsPowerShell`
- Some modules (e.g., older SCCM, certain AD modules) need WinPS compatibility mode

### SSH Remoting (PS7+)
- `New-PSSession -HostName server -UserName admin -KeyFilePath ~/.ssh/id_rsa`
- `Enter-PSSession -HostName server -UserName admin`
- Cross-platform — Windows → Linux, Linux → Windows, Linux → Linux
- Requires SSH server with PowerShell subsystem: `Subsystem powershell /usr/bin/pwsh -sshs`
- WinRM still works on Windows — SSH is for cross-platform scenarios
- No JEA support over SSH yet
