# Azure Pipelines Gotchas

Subtle behaviors and non-obvious interactions that catch even experienced pipeline authors.

## Expression Evaluation Timing

Azure Pipelines has **three** expression syntaxes with different evaluation times:

| Syntax | When Evaluated | Access To |
|---|---|---|
| `${{ variables.foo }}` | **Compile time** (template expansion) | Parameters, statically defined variables, predefined variables |
| `$(foo)` | **Runtime** (before each task) | All variables including dynamically set ones |
| `$[variables.foo]` | **Runtime** (before job/stage) | Runtime variables, used in `condition:` expressions |

**Gotcha:** A template expression `${{ }}` that references a variable set by a previous task will always get the *initial* value — it was already resolved before the pipeline ran. Use `$(var)` or `$[variables.var]` for dynamic values.

## Trigger Path Filter Interactions

`include` and `exclude` path filters interact non-obviously:
- If you specify only `exclude`, Azure Pipelines implicitly includes everything else
- If you specify both `include` and `exclude`, only included paths are considered, then excludes are subtracted
- Path filters are case-insensitive on Windows agents but case-sensitive on Linux agents
- PR triggers and CI triggers evaluate path filters independently

**Gotcha:** Adding `paths.exclude: ['docs/*']` without any `paths.include` means "trigger on everything except docs" — not "don't trigger on docs changes."

## Multi-Stage Execution Order

- **Stages** run sequentially by default (or per `dependsOn`)
- **Jobs within a stage** can run in parallel if they have no `dependsOn`
- **Steps within a job** always run sequentially

**Gotcha:** If two jobs in the same stage both modify the same resource without `dependsOn`, you get a race condition. Jobs in parallel don't share workspace state.

## Service Connection Scoping

Each service connection type has different authentication scopes:
- **Azure Resource Manager:** Scoped to subscription or resource group
- **Docker Registry:** Scoped to a single registry
- **Kubernetes:** Scoped to cluster + namespace
- **Generic:** No scope enforcement — entirely credential-dependent

**Gotcha:** An ARM service connection with subscription scope can't be narrowed at the pipeline level. Create separate connections per resource group for least-privilege.

## Variable Precedence Order

When the same variable name is defined at multiple levels, the highest-priority value wins:

1. **Queue-time variables** (set when manually running) — highest
2. **Pipeline-level variables** (YAML root `variables:`)
3. **Stage-level variables**
4. **Job-level variables**
5. **Task-level variables** (task `env:` or output vars) — lowest for definitions, but output vars override at runtime

**Gotcha:** A variable group linked at the pipeline level can be silently overridden by a queue-time variable with the same name.

## Environment Project Boundaries

Environments can only be created in a single Azure DevOps project. Cross-project environment sharing requires service connections or project-scoped permissions.

**Gotcha:** If you move a repo to another project, existing environment references break. Environments aren't portable across projects — you must recreate them and reconfigure approval checks.

## Multi-Repo Checkout Directory Behavior

With single-repo checkout, code lands in `$(Build.SourcesDirectory)`. With multi-repo checkout:
- **Self repo** → `$(Build.SourcesDirectory)/self-repo-name`
- **Other repos** → `$(Build.SourcesDirectory)/repo-name`

**Gotcha:** Scripts that use `$(Build.SourcesDirectory)` directly break with multi-repo checkout because the code is now one directory deeper. Use `$(Build.Repository.LocalPath)` for the self repo.

## Condition Evaluation Defaults

- Every step and job has an implicit `condition: succeeded()` — it only runs if all previous steps/jobs succeeded
- `failed()` must be explicitly specified; it doesn't run automatically
- `always()` runs regardless of prior outcome — including cancellation
- `canceled()` is separate from `failed()`

**Gotcha:** A cleanup step with `condition: always()` also runs when the pipeline is *canceled*, which may cause issues if resources are in a partially deployed state.

## YAML Schema and Pool Restrictions

Some YAML features are only available on specific pool types:
- `container:` jobs require Linux-based agents (hosted or self-hosted with Docker)
- VMSS scale-set pool features require Azure DevOps Services (not Server)
- `demands:` are only evaluated for self-hosted agent pools
- Microsoft-hosted agents don't support `demands:`

**Gotcha:** A pipeline that works on Microsoft-hosted agents may fail on self-hosted agents if the required tools aren't installed — there's no automatic tool provisioning.

## Template Parameter Defaults

Template parameter `default:` values are resolved at **compile time** (template expansion). They cannot reference runtime variables or output variables from other jobs.

```yaml
# This default is always 'Release' — it can't be a runtime variable
parameters:
  - name: configuration
    type: string
    default: Release  # Compile-time literal only
```

**Gotcha:** Setting `default: $(someVar)` will literally pass the string `$(someVar)` as the default, not the variable's value. Use `${{ variables.someVar }}` only if the variable is statically defined.

## Quick Reference

| Gotcha | Impact | Mitigation |
|---|---|---|
| Expression timing | Wrong variable values | Match syntax to evaluation phase |
| Path filter interaction | Unexpected triggers | Always pair include + exclude |
| Parallel jobs in a stage | Race conditions | Use `dependsOn` for shared resources |
| Service connection scope | Over-privileged access | Create per-resource-group connections |
| Variable precedence | Silent overrides | Audit queue-time variable settings |
| Environment boundaries | Broken cross-project refs | Plan environments per project |
| Multi-repo checkout paths | Broken script paths | Use `$(Build.Repository.LocalPath)` |
| Default condition | Cleanup steps skipped | Use `always()` for teardown |
| Pool-specific features | Works locally, fails in CI | Test on target pool type |
| Parameter defaults | Literal string instead of value | Only use compile-time expressions |
