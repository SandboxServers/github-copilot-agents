# Module Development

## Module Types
- **Script modules** (.psm1): Pure PowerShell. Most common. What you'll build 99% of the time.
- **Binary modules** (.dll): Compiled C# cmdlets. Use when you need .NET performance or complex type systems.
- **Manifest modules** (.psd1 only): Just a manifest pointing to nested modules or assemblies.

## Standard Folder Structure
```
MyModule/
├── MyModule.psd1           # Module manifest (required)
├── MyModule.psm1           # Root module (dot-sources Public/Private)
├── Public/                 # Exported functions (one file per function)
│   ├── Get-Something.ps1
│   ├── Set-Something.ps1
│   └── Remove-Something.ps1
├── Private/                # Internal helper functions (not exported)
│   ├── Convert-InternalFormat.ps1
│   └── Test-Prerequisite.ps1
├── Classes/                # PowerShell classes (if needed)
│   └── MyCustomType.ps1
├── Tests/                  # Pester tests
│   ├── Public/
│   │   └── Get-Something.Tests.ps1
│   └── Private/
│       └── Convert-InternalFormat.Tests.ps1
├── en-US/                  # Localized help (optional)
│   └── MyModule-help.xml
├── CHANGELOG.md
└── README.md
```

## Root Module (.psm1) Pattern
```powershell
# MyModule.psm1
# Dot-source all public and private functions
$publicFunctions  = @(Get-ChildItem -Path "$PSScriptRoot/Public/*.ps1" -ErrorAction SilentlyContinue)
$privateFunctions = @(Get-ChildItem -Path "$PSScriptRoot/Private/*.ps1" -ErrorAction SilentlyContinue)

foreach ($file in @($publicFunctions + $privateFunctions)) {
    try {
        . $file.FullName
    }
    catch {
        Write-Error "Failed to import function $($file.FullName): $_"
    }
}

# Export only public functions — private functions stay internal
Export-ModuleMember -Function $publicFunctions.BaseName
```

## Module Manifest (.psd1) Key Fields
```powershell
@{
    RootModule        = 'MyModule.psm1'
    ModuleVersion     = '1.2.0'
    GUID              = 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'  # New-Guid
    Author            = 'Your Name'
    CompanyName       = 'Your Org'
    Description       = 'One clear sentence about what this module does.'
    PowerShellVersion = '7.0'
    
    # Explicit is faster than wildcard — list every public function
    FunctionsToExport = @(
        'Get-Something'
        'Set-Something'
        'Remove-Something'
    )
    CmdletsToExport   = @()     # Empty = export none
    VariablesToExport  = @()     # Empty = export none
    AliasesToExport    = @()     # Empty = export none
    
    # Dependencies
    RequiredModules    = @(
        @{ ModuleName = 'Az.Accounts'; ModuleVersion = '2.0.0' }
        @{ ModuleName = 'Az.Resources'; ModuleVersion = '6.0.0' }
    )
    
    # Pre-release tag for SemVer pre-releases
    PrivateData = @{
        PSData = @{
            Tags         = @('Azure', 'Automation')
            LicenseUri   = 'https://github.com/org/repo/blob/main/LICENSE'
            ProjectUri   = 'https://github.com/org/repo'
            ReleaseNotes = 'See CHANGELOG.md'
            Prerelease   = ''  # Set to 'beta1', 'rc1' etc. for pre-releases
        }
    }
}
```

## Comment-Based Help (Required on Every Function)
```powershell
function Get-StaleServicePrincipal {
    <#
    .SYNOPSIS
        Finds Azure AD service principals that haven't been used recently.
    
    .DESCRIPTION
        Queries Microsoft Graph for service principals and checks their last
        sign-in activity. Returns those with no activity in the specified
        number of days. Requires Microsoft.Graph.Applications permissions.
    
    .PARAMETER DaysInactive
        Number of days of inactivity to qualify as stale. Default: 90.
    
    .PARAMETER ExcludeAppIds
        App IDs to exclude from the stale check (e.g., known long-lived SPs).
    
    .EXAMPLE
        Get-StaleServicePrincipal -DaysInactive 120

        Returns all service principals with no sign-in activity in 120+ days.
    
    .EXAMPLE
        Get-StaleServicePrincipal | Export-Csv -Path ./stale-sps.csv

        Exports stale service principals to CSV.
    
    .OUTPUTS
        PSCustomObject with properties: AppId, DisplayName, LastSignIn, DaysInactive
    
    .NOTES
        Requires: Microsoft.Graph.Applications module
        Permissions: Application.Read.All
    #>
    [CmdletBinding()]
    [OutputType([PSCustomObject])]
    param(
        [Parameter()]
        [int]$DaysInactive = 90,

        [Parameter()]
        [string[]]$ExcludeAppIds = @()
    )
    # Implementation...
}
```

## Versioning Strategy (SemVer)
- **MAJOR**: Breaking changes (renamed parameters, removed functions, changed output types)
- **MINOR**: New functions, new parameters, new features (backward-compatible)
- **PATCH**: Bug fixes, documentation updates (no API changes)
- Update manifest version: `Update-ModuleManifest -Path ./MyModule.psd1 -ModuleVersion '1.3.0'`
- Pre-release: Set `Prerelease = 'beta1'` in PrivateData.PSData

## Module Loading Mechanics
```powershell
# Explicit import
Import-Module MyModule -Force   # -Force reimports even if already loaded

# Auto-loading: PS auto-imports when you call an exported function
# Only works if module is in $env:PSModulePath

# Check module path
$env:PSModulePath -split [IO.Path]::PathSeparator

# #Requires at script level (enforces before execution)
#Requires -Modules @{ ModuleName = 'Az.Accounts'; ModuleVersion = '2.0.0' }
#Requires -Version 7.0
```

## Publishing

### To PSGallery
```powershell
# Test first
Test-ModuleManifest -Path ./MyModule/MyModule.psd1

# Publish
Publish-Module -Path ./MyModule -NuGetApiKey $apiKey -Repository PSGallery
```

### To Private Gallery (Azure Artifacts)
```powershell
# Register the feed
$registerParams = @{
    Name               = 'InternalFeed'
    SourceLocation     = 'https://pkgs.dev.azure.com/org/project/_packaging/feed/nuget/v2'
    PublishLocation    = 'https://pkgs.dev.azure.com/org/project/_packaging/feed/nuget/v2'
    InstallationPolicy = 'Trusted'
}
Register-PSRepository @registerParams

# Publish
$credential = Get-Credential   # PAT as password
Publish-Module -Path ./MyModule -Repository 'InternalFeed' -NuGetApiKey 'az' -Credential $credential
```

### CI/CD Publish Pipeline
```yaml
# Azure Pipelines: build → test → publish
trigger:
  tags:
    include: ['v*']

stages:
  - stage: Test
    jobs:
      - job: Pester
        steps:
          - pwsh: |
              $config = New-PesterConfiguration
              $config.Run.Path = './Tests'
              $config.CodeCoverage.Enabled = $true
              $config.CodeCoverage.CoveragePercentTarget = 80
              $results = Invoke-Pester -Configuration $config
              if ($results.FailedCount -gt 0) { exit 1 }
  
  - stage: Publish
    dependsOn: Test
    condition: succeeded()
    jobs:
      - job: PublishModule
        steps:
          - pwsh: |
              # Version from git tag
              $version = "$env:BUILD_SOURCEBRANCHNAME".TrimStart('v')
              Update-ModuleManifest -Path ./MyModule/MyModule.psd1 -ModuleVersion $version
              Publish-Module -Path ./MyModule -NuGetApiKey $(NuGetApiKey) -Repository PSGallery
```

## Dependency Conflict Resolution
- Pin versions in `RequiredModules` with `ModuleVersion` (minimum) or `RequiredVersion` (exact)
- Use `#Requires -Modules` at script level for additional enforcement
- For conflicting transitive dependencies, isolate with runspaces or separate processes
- Test module loading in a clean session: `pwsh -NoProfile -Command "Import-Module ./MyModule; Get-Module"`

## Progression: Script → Module
| Stage | Use When |
|---|---|
| Standalone script (.ps1) | One-off task, no reuse expected |
| Function library (.ps1 with functions) | Shared by 2-3 scripts, team use |
| Script module (.psm1 + .psd1) | Published internally, versioned, tested |
| Published module (PSGallery/feed) | Cross-team or public consumption |
