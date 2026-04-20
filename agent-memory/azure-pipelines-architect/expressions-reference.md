# Azure Pipelines Expression Reference

## Three Expression Syntaxes

Azure Pipelines has three distinct expression syntaxes evaluated at different times:

| Syntax | Evaluation Time | Context | Example |
|---|---|---|---|
| `${{ }}` | **Compile time** — during YAML parsing, before pipeline runs | Template parameters, literal YAML values | `${{ parameters.env }}` |
| `$[ ]` | **Runtime** — during job/stage execution | Conditions, variable definitions using dependencies | `$[dependencies.A.outputs['job.var']]` |
| `$(variableName)` | **Task execution** — macro expansion by the agent | Task inputs, script inline | `$(Build.BuildId)` |

### When to Use Which

```yaml
variables:
  staticVar: ${{ parameters.region }}           # Compile-time: baked into YAML before run
  dynamicVar: $[dependencies.Build.outputs['BuildJob.setVer.version']]  # Runtime: resolved during execution

steps:
  - script: echo $(staticVar) $(dynamicVar)     # Macro: expanded by agent at task time
```

## Compile-Time Expressions `${{ }}`

Evaluated during YAML parsing. The entire expression is replaced with its result before the pipeline runs. Available context: `parameters.*`, variables defined in YAML literals, `resources.*`.

```yaml
# Conditionally insert a stage at compile time
stages:
  - ${{ if eq(parameters.includeTests, true) }}:
    - stage: Test
      jobs:
        - job: RunTests
          steps:
            - script: dotnet test

  # Conditional variable assignment
  variables:
    ${{ if eq(parameters.environment, 'production') }}:
      serviceConnection: sc-prod
    ${{ else }}:
      serviceConnection: sc-nonprod
```

## Runtime Expressions `$[ ]`

Evaluated during pipeline execution. Used in `condition:` and `variables:` blocks when you need values from previous jobs/stages (dependencies).

```yaml
stages:
  - stage: A
    jobs:
      - job: A1
        steps:
          - bash: echo "##vso[task.setvariable variable=shouldDeploy;isOutput=true]true"
            name: check

  - stage: B
    dependsOn: A
    condition: and(succeeded(), eq(dependencies.A.outputs['A1.check.shouldDeploy'], 'true'))
    variables:
      deployFlag: $[stageDependencies.A.A1.outputs['check.shouldDeploy']]
    jobs:
      - job: B1
        steps:
          - script: echo "Deploy flag is $(deployFlag)"
```

## Built-In Functions — Complete Reference

### Comparison and Logical
| Function | Description | Example |
|---|---|---|
| `eq(a, b)` | Equals (case-insensitive for strings) | `eq(variables['Build.SourceBranch'], 'refs/heads/main')` |
| `ne(a, b)` | Not equals | `ne(variables['Build.Reason'], 'PullRequest')` |
| `gt(a, b)` | Greater than | `gt(variables['System.PullRequest.PullRequestId'], 0)` |
| `lt(a, b)` | Less than | `lt(counter(variables['Build.SourceBranch'], 0), 100)` |
| `ge(a, b)` | Greater than or equal | `ge(variables['buildCount'], 5)` |
| `le(a, b)` | Less than or equal | `le(length(variables['msg']), 500)` |
| `and(a, b, ...)` | Logical AND (short-circuits) | `and(succeeded(), eq(variables['deploy'], 'true'))` |
| `or(a, b, ...)` | Logical OR (short-circuits) | `or(eq(variables['env'], 'dev'), eq(variables['env'], 'test'))` |
| `not(a)` | Logical NOT | `not(canceled())` |

### Collection Functions
| Function | Description | Example |
|---|---|---|
| `contains(collection, item)` | String contains or array contains | `contains(variables['Build.SourceBranch'], 'release')` |
| `startsWith(str, prefix)` | String starts with | `startsWith(variables['Build.SourceBranch'], 'refs/heads/feature/')` |
| `endsWith(str, suffix)` | String ends with | `endsWith(variables['Build.SourceBranch'], '-hotfix')` |
| `in(item, a, b, ...)` | Item is one of the listed values | `in(variables['Build.Reason'], 'IndividualCI', 'BatchedCI')` |
| `notIn(item, a, b, ...)` | Item is NOT one of the listed values | `notIn(variables['Build.Reason'], 'Schedule', 'Manual')` |
| `containsValue(collection, val)` | Check if object/array contains a value | `containsValue(parameters.regions, 'eastus')` |

### String Functions
| Function | Description | Example |
|---|---|---|
| `format(fmt, arg0, ...)` | String formatting with `{0}`, `{1}` | `format('deploy-{0}-{1}', parameters.env, parameters.region)` |
| `join(separator, collection)` | Join array to string | `join(',', parameters.regions)` |
| `replace(str, old, new)` | Replace substring | `replace(variables['Build.SourceBranch'], 'refs/heads/', '')` |
| `split(str, delimiter)` | Split string to array | `split(variables['tags'], ',')` |
| `upper(str)` | Uppercase | `upper(parameters.environment)` |
| `lower(str)` | Lowercase | `lower(variables['Build.Repository.Name'])` |
| `trim(str)` | Trim whitespace | `trim(variables['userInput'])` |
| `length(collection)` | Length of string or array | `length(parameters.regions)` |

### Utility Functions
| Function | Description | Example |
|---|---|---|
| `coalesce(a, b, ...)` | First non-null/empty value | `coalesce(variables['customImage'], 'ubuntu-latest')` |
| `counter(prefix, seed)` | Auto-incrementing counter | `counter(variables['Build.SourceBranch'], 0)` |
| `convertToJson(value)` | Serialize to JSON string | `convertToJson(parameters.config)` |

## Type Casting Rules

- **Boolean evaluation**: Empty string `''`, `0`, `null`, and `false` are falsy; everything else is truthy
- **String to number**: Happens automatically in `gt()`, `lt()`, `ge()`, `le()` comparisons
- **All comparisons are case-insensitive** for strings in `eq()`, `ne()`, `contains()`, etc.

## Complex Expression Patterns

### Nested Function Calls
```yaml
condition: |
  and(
    succeeded(),
    or(
      eq(variables['Build.SourceBranch'], 'refs/heads/main'),
      startsWith(variables['Build.SourceBranch'], 'refs/heads/release/')
    ),
    ne(variables['skipDeploy'], 'true')
  )
```

### Cross-Job Output Variable in Condition
```yaml
jobs:
  - job: Validate
    steps:
      - bash: |
          if [[ -f "deploy.flag" ]]; then
            echo "##vso[task.setvariable variable=ready;isOutput=true]true"
          fi
        name: checker
  - job: Deploy
    dependsOn: Validate
    condition: eq(dependencies.Validate.outputs['checker.ready'], 'true')
    variables:
      readyFlag: $[dependencies.Validate.outputs['checker.ready']]
```

### Counter for Semantic Versioning
```yaml
variables:
  majorMinor: '2.1'
  patch: $[counter(variables['majorMinor'], 0)]  # Resets to 0 when majorMinor changes
  version: ${{ variables.majorMinor }}.$(patch)   # 2.1.0, 2.1.1, 2.1.2...
```

## Common Gotchas

| Gotcha | Problem | Fix |
|---|---|---|
| `${{ variables.foo }}` in condition | Variable not available at compile time | Use `$[variables['foo']]` for runtime |
| Runtime expression in template parameter | `$[ ]` can't be used in template params | Template params are compile-time only — use `${{ }}` |
| Comparing output variable to boolean | Output variables are always strings | Use `eq(dependencies.A.outputs['j.s.var'], 'true')` not `eq(..., true)` |
| Expression too long | Runtime limit on expression length | Break into multiple variables, combine in condition |
| `contains()` is case-insensitive | May match unintended substrings | Use `eq()` for exact matching |
| Missing quotes on string literals | `eq(var, main)` — `main` treated as variable name | Always quote: `eq(var, 'main')` |
