## Pester Testing

### Structure
```powershell
Describe 'Get-UserReport' {
    BeforeAll {
        . $PSScriptRoot/../src/Get-UserReport.ps1
        # One-time setup
    }
    BeforeEach {
        # Per-test setup
    }
    Context 'When user exists' {
        It 'Should return user display name' {
            # Arrange
            Mock Get-MgUser { return @{ DisplayName = 'Test User' } }
            # Act
            $result = Get-UserReport -UserId 'test@contoso.com'
            # Assert
            $result.DisplayName | Should -Be 'Test User'
        }
        It 'Should call Graph API exactly once' {
            Should -Invoke Get-MgUser -Times 1 -Exactly
        }
    }
    Context 'When user does not exist' {
        It 'Should return null' {
            Mock Get-MgUser { throw [Microsoft.Graph.Models.ODataErrors.ODataError]::new() }
            $result = Get-UserReport -UserId 'missing@contoso.com'
            $result | Should -BeNullOrEmpty
        }
    }
    AfterAll {
        # Cleanup
    }
}
```

### Key Assertions
- `Should -Be` / `-BeExactly` — value equality (case-insensitive / case-sensitive)
- `Should -BeNullOrEmpty` / `-Not -BeNullOrEmpty`
- `Should -HaveCount 5`
- `Should -Contain` (collection contains item) vs `-Match` (regex)
- `Should -Throw` / `-Throw -ExceptionType [System.InvalidOperationException]`
- `Should -Invoke` — verify mock was called with specific parameters

### Mocking
```powershell
Mock Invoke-RestMethod { return @{ value = @() } } -ParameterFilter { $Uri -like '*/users*' }
Mock Connect-MgGraph { }  # Suppress authentication in tests
Mock Write-Error { } -Verifiable  # Track that error was written
```
- Mock at the function level, not inside `It` blocks (unless overriding)
- Use `-ParameterFilter` for conditional mocks
- Use `-Verifiable` + `Should -InvokeVerifiable` for mandatory call verification
- Mock external dependencies: Graph SDK, Az cmdlets, REST calls, file system

### Test Organization
- File naming: `FunctionName.Tests.ps1`
- Tests folder mirrors source structure
- `BeforeAll` for dot-sourcing scripts, one-time setup
- `BeforeEach` for per-test isolation
- Test idempotency: each `It` block should be independent
- **Configuration**: `Invoke-Pester -Configuration @{ Run = @{ Path = './tests' }; Output = @{ Verbosity = 'Detailed' }; CodeCoverage = @{ Enabled = $true; Path = './src' } }`
