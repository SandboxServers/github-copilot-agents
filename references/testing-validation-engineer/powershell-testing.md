### PowerShell Testing

#### Static Analysis — PSScriptAnalyzer
PSScriptAnalyzer is the static code checker for PowerShell. It evaluates `.ps1`, `.psm1`, and `.psd1` files against best-practice rules and returns diagnostic results (errors and warnings).

| Capability | Usage |
|---|---|
| **Lint all scripts** | `Invoke-ScriptAnalyzer -Path ./src -Recurse` |
| **Specific rules** | `-IncludeRule PSAvoidUsingPlainTextForPassword, PSAvoidUsingInvokeExpression` |
| **Exclude rules** | `-ExcludeRule PSAvoidUsingWriteHost` |
| **Auto-fix formatting** | `Invoke-ScriptAnalyzer -Path ./src -Fix` |
| **CI exit codes** | `-EnableExit` — exits with count of error records |
| **Severity filtering** | `-Severity Error, Warning` |
| **Custom rules** | Write rules in PowerShell scripts or compile in C# assemblies |

**Key rules to enforce:**
- `PSAvoidUsingPlainTextForPassword` — no plaintext secrets
- `PSAvoidUsingConvertToSecureStringWithPlainText` — no insecure credential handling
- `PSAvoidUsingInvokeExpression` — no `Invoke-Expression` (injection risk)
- `PSUseShouldProcessForStateChangingFunctions` — state-changing functions should support `-WhatIf`
- `PSUseApprovedVerbs` — follow PowerShell verb conventions
- `PSAvoidUsingPositionalParameters` — explicit parameter names for readability
- `PSUseCompatibleCommands` / `PSUseCompatibleTypes` — cross-edition compatibility (Desktop vs Core)

**Suppression when justified:**
```powershell
[System.Diagnostics.CodeAnalysis.SuppressMessageAttribute('PSAvoidUsingWriteHost', '')]
param()
```

#### Pester — Unit and Integration Testing

Pester is the PowerShell testing framework. Ships with Windows 10+ and is the recommended framework per Microsoft guidelines.

**Test Structure:**
```powershell
Describe 'Get-UserConfig' {
    Context 'When config file exists' {
        BeforeAll {
            Mock Get-Content { return '{"setting": "value"}' }
        }

        It 'Should return parsed configuration' {
            $result = Get-UserConfig -Path 'config.json'
            $result.setting | Should -Be 'value'
        }

        It 'Should call Get-Content with the correct path' {
            Should -Invoke Get-Content -ParameterFilter { $Path -eq 'config.json' }
        }
    }

    Context 'When config file is missing' {
        BeforeAll {
            Mock Get-Content { throw [System.IO.FileNotFoundException]::new() }
        }

        It 'Should throw a meaningful error' {
            { Get-UserConfig -Path 'missing.json' } | Should -Throw '*not found*'
        }
    }
}
```

**Key Pester Capabilities:**
| Feature | Purpose |
|---|---|
| `Mock` | Replace cmdlet behavior — mock Azure cmdlets, external APIs, filesystem calls |
| `Should -Invoke` | Assert a mock was called with expected parameters and count |
| `BeforeAll` / `BeforeEach` | Setup — module imports, mock definitions, test data |
| `AfterAll` / `AfterEach` | Teardown — cleanup resources, reset state |
| `InModuleScope` | Test private/internal module functions |
| `-Tag` | Categorize tests (Unit, Integration, Slow) for selective execution |
| Code coverage | `Invoke-Pester -CodeCoverage ./src/*.ps1` — track which lines are exercised |

**CI Pipeline Integration:**
```powershell
$pesterConfig = New-PesterConfiguration
$pesterConfig.Run.Path = './tests'
$pesterConfig.Run.Exit = $true
$pesterConfig.TestResult.Enabled = $true
$pesterConfig.TestResult.OutputFormat = 'NUnitXml'
$pesterConfig.TestResult.OutputPath = './testresults/pester-results.xml'
$pesterConfig.CodeCoverage.Enabled = $true
$pesterConfig.CodeCoverage.Path = './src/*.ps1'
$pesterConfig.CodeCoverage.OutputPath = './testresults/coverage.xml'
Invoke-Pester -Configuration $pesterConfig
```

**Mocking Azure Cmdlets (Common Patterns):**
```powershell
Mock Get-AzResource { return @{ Name = 'test-rg'; ResourceType = 'Microsoft.Resources/resourceGroups' } }
Mock New-AzResourceGroupDeployment { return @{ ProvisioningState = 'Succeeded' } }
Mock Invoke-MgGraphRequest { return @{ value = @() } }
```

**What to Test in PowerShell Automation:**
- **Parameter validation** — mandatory params, ValidateSet, ValidatePattern, pipeline input
- **Error handling** — try/catch behavior, ErrorAction, terminating vs non-terminating errors
- **Idempotency** — running the script twice produces the same result
- **Azure interactions** — mock Az module cmdlets, validate correct parameters are passed
- **Credential handling** — secrets stay as SecureString, no plaintext in logs or output
- **Cross-platform** — test on both PowerShell 7 (Core) and Windows PowerShell 5.1 where applicable

#### PSRule for Azure
Rule-based validation of ARM/Bicep templates against Azure Well-Architected Framework best practices. Required tooling for Bicep AVM modules (BCPNFR11).

| Feature | Purpose |
|---|---|
| Template validation | Validate ARM JSON / Bicep against Azure best practices without deployment |
| Custom rules | Write organization-specific rules in PowerShell or YAML |
| Baselines | Apply different rule sets per environment (dev vs prod) |
| CI integration | `Assert-PSRule -InputPath ./templates -Module PSRule.Rules.Azure` |

#### Post-Deployment Pester Tests
Used by Azure Verified Modules (BCPNFR16) for validating deployed resources:
- Test files named `*.tests.ps1` in `e2e` test folders
- Receive `$TestInputData` hashtable with deployment outputs (resource IDs, endpoints)
- Validate deployed resources match expected configuration using Az cmdlets
- Run after deployment completes, before cleanup
