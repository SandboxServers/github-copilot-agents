## Module Development

### Module Structure
```
MyModule/
├── MyModule.psd1          # Module manifest
├── MyModule.psm1          # Root module (script module)
├── Public/                # Exported functions
│   ├── Get-Something.ps1
│   └── Set-Something.ps1
├── Private/               # Internal helper functions
│   └── Convert-Internal.ps1
└── Tests/
    └── Get-Something.Tests.ps1
```

### Module Manifest (psd1) Key Fields
```powershell
@{
    RootModule        = 'MyModule.psm1'
    ModuleVersion     = '1.0.0'
    CompatiblePSEditions = @('Core','Desktop')  # or just 'Core' for PS7-only
    GUID              = '<generate-new-guid>'
    Author            = 'Team Name'
    FunctionsToExport = @('Get-Something','Set-Something')  # Never use '*' in production
    CmdletsToExport   = @()
    VariablesToExport  = @()
    AliasesToExport    = @()
    RequiredModules    = @('Az.Accounts')
    PrivateData = @{
        PSData = @{
            Tags       = @('Azure','Automation')
            ProjectUri = 'https://github.com/org/module'
        }
    }
}
```

### Script Module Pattern (psm1)
```powershell
# Dot-source all public and private functions
$Public  = @(Get-ChildItem -Path "$PSScriptRoot/Public/*.ps1" -ErrorAction SilentlyContinue)
$Private = @(Get-ChildItem -Path "$PSScriptRoot/Private/*.ps1" -ErrorAction SilentlyContinue)
foreach ($import in @($Public + $Private)) {
    try { . $import.FullName }
    catch { Write-Error "Failed to import $($import.FullName): $_" }
}
Export-ModuleMember -Function $Public.BaseName
```

### When Script → Function → Module → Published Module
- **Script**: one-off task, no reuse expected
- **Function**: reusable logic, called from multiple scripts in same project
- **Module**: shared across projects or teams, needs versioning
- **Published Module**: organization-wide tool, needs gallery hosting, semantic versioning, changelog

### Private Gallery Hosting
- **Azure Artifacts**: NuGet feed, integrates with ADO pipelines
- **GitHub Packages**: NuGet feed, integrates with GitHub Actions
- **NuGet server**: self-hosted, `Register-PSRepository -SourceLocation`
- **File share**: simple `Register-PSRepository -SourceLocation \\server\share\repo`
- Publish: `Publish-Module -Name MyModule -Repository PrivateGallery -NuGetApiKey $key`

## Comment-Based Help

### Pattern
```powershell
function Get-UserLicense {
    <#
    .SYNOPSIS
        Retrieves Microsoft 365 license assignments for a user.

    .DESCRIPTION
        Queries Microsoft Graph API to retrieve all license SKUs assigned to
        the specified user. Returns both friendly names and SKU IDs.

    .PARAMETER UserId
        The UPN or object ID of the user.

    .PARAMETER IncludeDisabledPlans
        When specified, includes disabled service plans in the output.

    .EXAMPLE
        Get-UserLicense -UserId 'user@contoso.com'

        Returns all active license assignments for the specified user.

    .EXAMPLE
        Get-UserLicense -UserId 'user@contoso.com' -IncludeDisabledPlans

        Returns license assignments including disabled service plans.

    .OUTPUTS
        PSCustomObject with properties: UserId, SkuId, SkuPartNumber, DisabledPlans

    .NOTES
        Requires Microsoft.Graph.Users module and User.Read.All permission.
    #>
    [CmdletBinding()]
    param(
        [Parameter(Mandatory, ValueFromPipeline, ValueFromPipelineByPropertyName)]
        [ValidateNotNullOrEmpty()]
        [string]$UserId,

        [switch]$IncludeDisabledPlans
    )
    # ... implementation
}
```
