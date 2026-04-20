# Testing & Integration Migration: ADO → GitHub Actions

## Test Result Publishing

### ADO PublishTestResults@2 → GHA Alternatives

ADO provides `PublishTestResults@2` which natively renders test results in the pipeline's Test tab with
filtering, grouping, and historical trend analysis. GitHub Actions has no built-in test result rendering —
you must use community actions that create **check run annotations** or **PR comments**.

#### dorny/test-reporter (Recommended Primary)
- Supports: JUnit, xUnit, NUnit, TRX, Mocha JSON, Jest JSON
- Creates check runs with inline file annotations on failures
- Summary shows pass/fail/skip/error counts with duration
- Works for both push and pull_request events

```yaml
- name: Publish Test Results
  uses: dorny/test-reporter@v1
  if: always()                # Run even if tests fail
  with:
    name: 'Test Results'
    path: '**/TestResults/*.trx'
    reporter: dotnet-trx       # Also: java-junit, jest-junit, etc.
    fail-on-error: false       # Don't fail the workflow on test failure
```

#### EnricoMi/publish-unit-test-result-action
- Best for PR workflows — adds a comment with test result summary
- Compares results against base branch (shows new failures)
- Supports JUnit XML, NUnit XML, XUnit XML, TRX

```yaml
- name: Publish Unit Test Results
  uses: EnricoMi/publish-unit-test-result-action@v2
  if: always()
  with:
    files: '**/test-results/**/*.xml'
    check_name: 'Unit Test Results'
    comment_mode: always        # always | changes | off
```

#### mikepenz/action-junit-report
- Lightweight — JUnit XML only
- Creates check run annotations for failures
- Good when you only need JUnit format support

```yaml
- name: JUnit Report
  uses: mikepenz/action-junit-report@v4
  if: always()
  with:
    report_paths: '**/build/test-results/**/*.xml'
    include_passed: true
```

## Code Coverage Publishing

### ADO PublishCodeCoverageResults@2 → GHA Alternatives

ADO renders coverage in the pipeline's Code Coverage tab with file-level detail. GHA has no native
coverage rendering — use external services or step summaries.

#### codecov/codecov-action (Most Popular)
- Supports: Cobertura XML, LCOV, JaCoCo XML, Clover, gcov
- PR comments with coverage diff and file-level annotations
- Historical tracking, coverage gates, and badge generation
- Requires Codecov account (free for public repos)

```yaml
- name: Upload Coverage to Codecov
  uses: codecov/codecov-action@v4
  with:
    files: ./coverage/cobertura.xml
    flags: unittests
    fail_ci_if_error: true
    token: ${{ secrets.CODECOV_TOKEN }}   # Required for private repos
```

#### irongut/CodeCoverageReport
- No external service needed — renders coverage in GITHUB_STEP_SUMMARY
- Supports Cobertura XML
- Good for teams that don't want external dependencies

```yaml
- name: Code Coverage Summary
  uses: irongut/CodeCoverageSummary@v1.3.0
  with:
    filename: coverage/cobertura.xml
    format: markdown
    output: both               # console and file
    badge: true
    thresholds: '60 80'        # warning and failure thresholds
```

#### Direct Upload to SonarCloud or Coveralls
- `coverallsapp/github-action` for Coveralls integration
- SonarCloud analysis includes coverage (see SonarQube section below)

## Azure Test Plans — No GitHub Equivalent

ADO Azure Test Plans provides manual test case management, exploratory testing, and test plan execution
tracking. **GitHub has no equivalent feature.** Options:

| Strategy | Complexity | Notes |
|---|---|---|
| Keep Azure Test Plans (hybrid) | Low | Access via ADO; link to GitHub repos via integration |
| Migrate to TestRail | Medium | Full test management; import via API |
| Migrate to Zephyr Scale | Medium | Jira ecosystem; strong if also migrating PM |
| Migrate to qTest | Medium | Enterprise-grade alternative |
| GitHub Issues for lightweight tracking | Low | Use issue templates + labels; limited for formal test plans |

**Hybrid approach is recommended** — Azure Test Plans can reference GitHub repos and there is no
immediate urgency to migrate manual test management.

## SonarQube / SonarCloud Migration

### ADO: SonarQube Extension Tasks → GHA: SonarSource Actions

```yaml
# ADO pattern                         # GHA equivalent
# SonarQubePrepare@5                  # sonarsource/sonarqube-scan-action
# SonarQubeAnalyze@5                  #   (or sonarcloud-github-action for SonarCloud)
# SonarQubePublish@5                  # Results posted as PR decoration

# GHA SonarCloud example:
- name: SonarCloud Scan
  uses: SonarSource/sonarcloud-github-action@v3
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
  with:
    args: >
      -Dsonar.projectKey=my-project
      -Dsonar.organization=my-org
      -Dsonar.sources=src/
      -Dsonar.tests=tests/
      -Dsonar.coverage.exclusions=**/*Test*/**
```

## Code Analysis: ADO Tasks → GitHub Native

### GitHub CodeQL (GitHub Advanced Security)

Replaces ADO code analysis tasks with native GHAS integration:

```yaml
- name: Initialize CodeQL
  uses: github/codeql-action/init@v3
  with:
    languages: csharp, javascript   # Supported: cpp, csharp, go, java, javascript, python, ruby, swift
    queries: security-and-quality   # security-extended, security-and-quality

- name: Build                        # Required for compiled languages
  run: dotnet build

- name: Perform CodeQL Analysis
  uses: github/codeql-action/analyze@v3
```

**GHAS also provides:** Secret scanning, push protection, Dependabot alerts/updates — features that
ADO either lacks or provides via extensions.

## Test Result Visualization Differences

| Feature | ADO | GitHub Actions |
|---|---|---|
| Native test tab | Yes — dedicated Test tab per run | No — use check annotations |
| Historical trends | Built-in test analytics | External tools (Codecov, Datadog) |
| Flaky test detection | Built-in flaky test tracking | No native support — use Datadog CI, BuildPulse, or Trunk |
| Test grouping | By suite, outcome, file | By check run (action-dependent) |
| PR impact analysis | Shows tests affected by changes | Via EnricoMi action or external |
| Inline failure annotations | Via Test tab drill-down | Via check run annotations (dorny) |

## Performance Testing Migration

ADO deprecated its built-in load testing. Current path:
- **Azure Load Testing** — use `azure/load-testing@v1` action in GHA
- **k6** — `grafana/k6-action@v0.3` for load testing in workflows
- **JMeter** — run via container action or self-hosted runner

```yaml
- name: Run Azure Load Test
  uses: azure/load-testing@v1
  with:
    loadTestConfigFile: 'loadtest/config.yaml'
    loadTestResource: 'my-load-test-resource'
    resourceGroup: 'my-rg'
```

## Migration Checklist for Testing Integration

1. **Inventory all ADO test tasks** — PublishTestResults, PublishCodeCoverage, VSTest, Azure Test Plans
2. **Map test result formats** — identify TRX, JUnit, NUnit, xUnit output across projects
3. **Choose primary test reporter action** — dorny/test-reporter for most teams
4. **Choose coverage strategy** — Codecov for full tracking, irongut for no-external-service
5. **Migrate SonarQube/SonarCloud config** — update project keys, create GHA secrets for tokens
6. **Enable GitHub Advanced Security** — CodeQL, secret scanning, Dependabot
7. **Plan Azure Test Plans approach** — hybrid or third-party migration
8. **Set up flaky test tracking** — evaluate BuildPulse, Trunk, or Datadog CI Visibility
9. **Validate test gate behavior** — ensure required status checks block merges on test failure
10. **Compare outputs** — run ADO and GHA in parallel, verify equivalent test visibility
