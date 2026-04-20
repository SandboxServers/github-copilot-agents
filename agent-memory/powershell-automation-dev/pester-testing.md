# Pester Testing

## Pester v5 Fundamentals

Pester v5 uses a two-phase model: **Discovery** (finds tests) and **Run** (executes them). Code at the top of `Describe`/`Context` blocks runs during Discovery, NOT during Run — this catches many v4 migration bugs. All setup code must go inside `BeforeAll`, `BeforeEach`, or `It` blocks.

## Test Structure

```powershell
BeforeAll {
    # Runs ONCE before all tests in this file
    # Import the module or dot-source the function under test
    . $PSCommandPath.Replace('.Tests.ps1', '.ps1')
}

Describe 'Get-UserStatus' {
    BeforeAll {
        # Runs once before all tests in this Describe
        $mockUser = @{ Name = 'TestUser'; Enabled = $true }
    }

    BeforeEach {
        # Runs before EACH It block
        $result = $null
    }

    Context 'When user exists' {
        It 'Should return the user object' {
            # Arrange is in BeforeAll/BeforeEach
            # Act
            $result = Get-UserStatus -Name 'TestUser'
            # Assert
            $result | Should -Not -BeNullOrEmpty
        }

        It 'Should have correct name' {
            $result = Get-UserStatus -Name 'TestUser'
            $result.Name | Should -Be 'TestUser'
        }
    }

    AfterAll {
        # Cleanup after all tests in this Describe
    }
}
```

## Key Assertions (Should)

```powershell
# Equality
$value | Should -Be 'expected'              # Case-insensitive string comparison
$value | Should -BeExactly 'Expected'       # Case-sensitive
$value | Should -BeNullOrEmpty
$value | Should -Not -BeNullOrEmpty

# Numeric
$count | Should -BeGreaterThan 0
$count | Should -BeLessOrEqual 100
$count | Should -BeIn @(1, 2, 3)

# Collections
$array | Should -HaveCount 5
$array | Should -Contain 'item'             # Collection contains value
'string' | Should -BeLike '*pattern*'        # Wildcard match
'string' | Should -Match '^str'              # Regex match

# Boolean
$result | Should -BeTrue
$result | Should -BeFalse
$result | Should -BeOfType [PSCustomObject]

# Exceptions
{ throw 'boom' } | Should -Throw
{ throw 'boom' } | Should -Throw -ExpectedMessage 'boom'
{ throw [System.IO.FileNotFoundException]::new() } | Should -Throw -ExceptionType ([System.IO.FileNotFoundException])

# Command invocation
Should -Invoke -CommandName 'Get-AzResource' -Times 1 -Exactly
Should -Invoke -CommandName 'Remove-AzResource' -Times 0
```

## Mocking

```powershell
Describe 'Remove-StaleUser' {
    BeforeAll {
        . $PSCommandPath.Replace('.Tests.ps1', '.ps1')
    }

    Context 'When stale users exist' {
        BeforeAll {
            # Mock an external dependency
            Mock Get-MgUser {
                return @(
                    [PSCustomObject]@{ Id = '1'; DisplayName = 'OldUser'; SignInActivity = @{ LastSignInDateTime = (Get-Date).AddDays(-180) } }
                )
            }

            # Mock the destructive action — don't actually delete
            Mock Remove-MgUser { }

            $result = Remove-StaleUser -DaysInactive 90
        }

        It 'Should query Graph for users' {
            Should -Invoke -CommandName 'Get-MgUser' -Times 1 -Exactly
        }

        It 'Should remove the stale user' {
            Should -Invoke -CommandName 'Remove-MgUser' -Times 1 -Exactly -ParameterFilter {
                $UserId -eq '1'
            }
        }
    }

    Context 'When no stale users exist' {
        BeforeAll {
            Mock Get-MgUser { return @() }
            Mock Remove-MgUser { }
            $result = Remove-StaleUser -DaysInactive 90
        }

        It 'Should not remove anyone' {
            Should -Invoke -CommandName 'Remove-MgUser' -Times 0
        }
    }
}
```

### Mock with ParameterFilter

```powershell
# Different return values based on parameters
Mock Get-AzKeyVaultSecret {
    return [PSCustomObject]@{ SecretValue = (ConvertTo-SecureString 'value1' -AsPlainText -Force) }
} -ParameterFilter { $Name -eq 'Secret1' }

Mock Get-AzKeyVaultSecret {
    return [PSCustomObject]@{ SecretValue = (ConvertTo-SecureString 'value2' -AsPlainText -Force) }
} -ParameterFilter { $Name -eq 'Secret2' }
```

## Data-Driven Tests

```powershell
Describe 'ConvertTo-Slug' {
    It 'Should convert "<Input>" to "<Expected>"' -ForEach @(
        @{ Input = 'Hello World';    Expected = 'hello-world' }
        @{ Input = 'Test  Spaces';   Expected = 'test-spaces' }
        @{ Input = 'UPPER-case';     Expected = 'upper-case' }
        @{ Input = 'special!@#chars'; Expected = 'specialchars' }
    ) {
        ConvertTo-Slug -Text $Input | Should -Be $Expected
    }
}
```

## TestDrive and TestRegistry

```powershell
Describe 'Export-Report' {
    It 'Should create the CSV file' {
        # TestDrive: is a temporary PSDrive scoped to the Describe/Context block
        # Automatically cleaned up when the block ends
        Export-Report -Path "TestDrive:\report.csv" -Data @{ Name = 'Test' }
        "TestDrive:\report.csv" | Should -Exist
    }

    It 'Should write correct content' {
        Export-Report -Path "TestDrive:\report.csv" -Data @{ Name = 'Test' }
        $content = Get-Content "TestDrive:\report.csv"
        $content | Should -Contain '"Test"'
    }
}

# TestRegistry: same concept for Windows registry
Describe 'Set-AppConfig' {
    It 'Should create registry key' {
        Set-AppConfig -Path "TestRegistry:\Software\MyApp" -Name 'Setting1' -Value 'test'
        Get-ItemProperty -Path "TestRegistry:\Software\MyApp" -Name 'Setting1' |
            Select-Object -ExpandProperty Setting1 |
            Should -Be 'test'
    }
}
```

## InModuleScope (Testing Private Functions)

```powershell
# When a function is NOT exported from a module, use InModuleScope
Describe 'Private helper function' {
    InModuleScope 'MyModule' {
        It 'Should format the name correctly' {
            # Format-InternalName is a private function in MyModule
            $result = Format-InternalName -First 'John' -Last 'Doe'
            $result | Should -Be 'Doe, John'
        }
    }
}
```

## Tag-Based Test Filtering

```powershell
Describe 'Database Operations' -Tag 'Integration', 'Database' {
    It 'Should connect to test database' -Tag 'Slow' {
        # ...
    }
}

# Run only specific tags
# Invoke-Pester -Tag 'Unit'
# Invoke-Pester -ExcludeTag 'Integration', 'Slow'
```

## Pester Configuration Object

```powershell
$pesterConfig = New-PesterConfiguration
$pesterConfig.Run.Path = './Tests'
$pesterConfig.Run.PassThru = $true
$pesterConfig.Output.Verbosity = 'Detailed'

# Code coverage
$pesterConfig.CodeCoverage.Enabled = $true
$pesterConfig.CodeCoverage.Path = @('./Public/*.ps1', './Private/*.ps1')
$pesterConfig.CodeCoverage.CoveragePercentTarget = 80
$pesterConfig.CodeCoverage.OutputFormat = 'JaCoCo'
$pesterConfig.CodeCoverage.OutputPath = './coverage.xml'

# CI/CD output (NUnit XML for pipeline reporting)
$pesterConfig.TestResult.Enabled = $true
$pesterConfig.TestResult.OutputFormat = 'NUnitXml'
$pesterConfig.TestResult.OutputPath = './testresults.xml'

$results = Invoke-Pester -Configuration $pesterConfig

# Fail the build if tests failed
if ($results.FailedCount -gt 0) {
    throw "$($results.FailedCount) test(s) failed"
}
```

## CI/CD Integration

```yaml
# Azure Pipelines example
- task: PowerShell@2
  displayName: 'Run Pester Tests'
  inputs:
    targetType: inline
    pwsh: true
    script: |
      $config = New-PesterConfiguration
      $config.Run.Path = './Tests'
      $config.TestResult.Enabled = $true
      $config.TestResult.OutputPath = '$(Build.ArtifactStagingDirectory)/testresults.xml'
      $config.TestResult.OutputFormat = 'NUnitXml'
      $config.CodeCoverage.Enabled = $true
      $config.CodeCoverage.OutputPath = '$(Build.ArtifactStagingDirectory)/coverage.xml'
      Invoke-Pester -Configuration $config

- task: PublishTestResults@2
  inputs:
    testResultsFormat: NUnit
    testResultsFiles: '$(Build.ArtifactStagingDirectory)/testresults.xml'
```

## Test Organization Best Practices

- One `.Tests.ps1` per function/script — `Get-UserStatus.Tests.ps1` tests `Get-UserStatus.ps1`
- Mirror folder structure: `Public/Get-UserStatus.ps1` → `Tests/Public/Get-UserStatus.Tests.ps1`
- Describe = function name, Context = scenario, It = specific behavior
- Name tests as assertions: "Should return empty array when no users found"
- Keep tests independent — no test should depend on another test's output
- Mock all external dependencies — Azure, Graph, file system, databases
