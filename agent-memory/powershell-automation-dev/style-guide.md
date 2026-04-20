# PowerShell Style Guide

## Philosophy

- Write for the reader, not the writer. The reader might be a junior admin at 2am, another AI agent parsing your output, or you in 6 months with no memory of writing this.
- Plain English variable names, function names, and comments. `$userMailboxes` not `$mbxs`. `Get-StaleServicePrincipal` not `Get-StaleSP`.
- Never sacrifice readability for cleverness. A three-line `if`/`else` beats a one-line ternary if the ternary requires mental parsing.
- Idempotency is non-negotiable. Every script must be safe to run twice. Check state before changing state. If it's already done, say so and move on.

## Naming Conventions

- Functions: Verb-Noun (approved verbs from `Get-Verb`). No custom verbs.
- Variables: PascalCase for parameters, camelCase for local variables. `$UserPrincipalName`, `$retryCount`.
- Boolean parameters: Use `$Force`, `$WhatIf`, `$Confirm` — not `$DoIt`, `$ShouldRun`.
- Script files: Verb-Noun.ps1 matching the primary function inside.
- Module folders: PascalCase matching module name.

## Commenting Standards

```powershell
# GOOD: Explains WHY, not what
# Retry because Graph API returns 429 during tenant-wide operations
Invoke-MgGraphRequest @params

# BAD: Restates the code
# Invoke the Graph request
Invoke-MgGraphRequest @params

# GOOD: Block comment for complex logic
<#
    We check for existing role assignments before creating new ones
    because Azure RBAC returns a conflict error (409) on duplicates
    and there's no -Force parameter on New-AzRoleAssignment.
#>

# BAD: No comment on non-obvious logic
$assignments | Where-Object { $_.RoleDefinitionName -eq $roleName }
```

- Every function gets comment-based help. No exceptions.
- Every `catch` block gets a comment explaining what we're catching and why.
- Every `$using:` variable in `-Parallel` gets a comment if non-obvious.
- Inline comments on the same line are acceptable for short clarifications.
- Don't comment obvious PowerShell — `# Loop through users` above `foreach ($user in $users)` is noise.

## Code Structure

- Splatting is mandatory for any command with 3+ parameters. No 200-character one-liners.
- One function per file in modules (Public/Private separation).
- `[CmdletBinding()]` on every function. No exceptions.
- `SupportsShouldProcess` on every function that changes state.
- Parameter sets over boolean flags when parameters are mutually exclusive.
- `[OutputType()]` on every function.

### Splatting Example

```powershell
# GOOD
$mailboxParams = @{
    Identity            = $UserPrincipalName
    Type                = 'Regular'
    RetentionPolicy     = 'Default MRM Policy'
    ErrorAction         = 'Stop'
}
Set-Mailbox @mailboxParams

# BAD
Set-Mailbox -Identity $UserPrincipalName -Type Regular -RetentionPolicy 'Default MRM Policy' -ErrorAction Stop
```

## Error Handling Rules

- Every `try` block has a `catch` with a typed exception where possible.
- Never use bare `catch { }` — always log, re-throw, or handle meaningfully.
- Use `-ErrorAction Stop` inside `try` blocks to catch non-terminating errors.
- Use `$PSCmdlet.ThrowTerminatingError()` with a constructed `ErrorRecord` for function-level errors — not bare `throw "message"`.
- `-ErrorVariable` for operations where you want to continue but track failures.
- Include the original exception in any re-thrown error (`$_.Exception`).

## Logging Standards

- Use `Write-Verbose` for operational step tracking (what the script is doing).
- Use `Write-Information` for structured output that calling scripts can capture.
- Use `Write-Warning` for non-fatal issues the user should know about.
- Use `Write-Error` for recoverable failures — never use it for terminating conditions.
- Never use `Write-Host` in functions/modules — it bypasses the pipeline and can't be captured.
- Timestamps in verbose output: `Write-Verbose "[$((Get-Date).ToString('HH:mm:ss'))] Processing $item"`
- Structured logging for automation:

```powershell
function Write-Log {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Message,
        [ValidateSet('Information', 'Warning', 'Error')]
        [string]$Level = 'Information'
    )
    $logEntry = [PSCustomObject]@{
        Timestamp = Get-Date -Format 'o'
        Level     = $Level
        Message   = $Message
        Script    = $MyInvocation.ScriptName
        Line      = $MyInvocation.ScriptLineNumber
    }
    $logEntry | Export-Csv -Path $script:LogPath -Append -NoTypeInformation
    switch ($Level) {
        'Warning'     { Write-Warning $Message }
        'Error'       { Write-Error $Message }
        default       { Write-Verbose $Message }
    }
}
```

## Idempotency Patterns

- Check before act. Always.

```powershell
# GOOD: Idempotent resource creation
$existingGroup = Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue
if ($existingGroup) {
    Write-Verbose "Resource group '$ResourceGroupName' already exists — skipping creation"
} else {
    Write-Verbose "Creating resource group '$ResourceGroupName'"
    New-AzResourceGroup -Name $ResourceGroupName -Location $Location
}

# BAD: Crashes on second run
New-AzResourceGroup -Name $ResourceGroupName -Location $Location
```

- Report what you did, what you skipped, and what failed. Every time.
- Return consistent output objects regardless of whether action was taken or skipped.
- Use `-WhatIf` to let users preview changes before applying.

## Output Standards

- Functions return objects, not strings. Use `[PSCustomObject]` with meaningful properties.
- Include a `Status` property on output objects: `Created`, `AlreadyExists`, `Updated`, `Failed`, `Skipped`.
- Never mix `Write-Output` with `Write-Host` — pick one stream strategy and stick with it.
- Suppress unwanted output with `[void]` or `$null =` — never let `.Add()` return values pollute the pipeline.
- Pipeline-friendly output: one object per item processed, not a summary object at the end.

## Security Rules

- Never embed credentials in scripts. Period.
- Never use `ConvertTo-SecureString -AsPlainText` with hardcoded strings.
- Use `SecretManagement` module for local development.
- Use managed identity or workload identity federation in cloud environments.
- Validate all user input — `[ValidatePattern()]`, `[ValidateSet()]`, `[ValidateScript()]`.
- Never use `Invoke-Expression`. If you think you need it, you're wrong.
- Use `-Credential` parameter, never prompt for passwords inline.

## What Not To Do

- Don't use aliases in scripts. `gci`, `%`, `?` are for interactive use. Scripts use `Get-ChildItem`, `ForEach-Object`, `Where-Object`.
- Don't use `Write-Host` in reusable code.
- Don't suppress errors globally with `$ErrorActionPreference = 'SilentlyContinue'` at script scope.
- Don't use positional parameters in scripts. Always use named parameters.
- Don't use `+=` on arrays in loops. Use `[System.Collections.Generic.List[object]]`.
- Don't use `Select-Object *` in production code. Be explicit about properties.
- Don't use `Format-*` cmdlets in functions that return data — formatting destroys objects.
