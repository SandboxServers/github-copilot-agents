# GitHub Actions Expressions and Contexts — Deep Reference

## Expression Syntax

Expressions use `${{ <expression> }}` syntax. In `if:` conditions, `${{ }}` is optional — GitHub auto-wraps the value. Exception: expressions starting with `!` MUST use `${{ }}` because YAML treats bare `!` as a tag.

```yaml
steps:
  - run: echo "SHA is ${{ github.sha }}"

  # if: auto-wrapped — these are equivalent:
  - if: github.ref == 'refs/heads/main'
  - if: ${{ github.ref == 'refs/heads/main' }}

  # MUST wrap with ${{ }} when using !
  - if: ${{ !cancelled() }}    # correct
  # - if: !cancelled()         # YAML parse error
```

## All Contexts with Key Properties

### github Context

```yaml
github.event_name        # push, pull_request, workflow_dispatch, etc.
github.event             # full webhook payload (differs per trigger)
github.ref               # refs/heads/main, refs/tags/v1.0, refs/pull/123/merge
github.ref_name          # main, v1.0, 123/merge (short form)
github.ref_type          # branch or tag
github.ref_protected     # true if branch protection rules apply
github.sha               # commit SHA triggering the workflow
github.actor             # user who triggered the event
github.triggering_actor  # user who initiated the workflow run
github.repository        # owner/repo
github.repository_owner  # owner
github.run_id            # unique ID for the workflow run
github.run_number        # sequential number per workflow (increments on re-run)
github.run_attempt       # retry attempt number (1-based)
github.workflow          # workflow name
github.job               # current job ID
github.token             # automatic GITHUB_TOKEN
github.server_url        # https://github.com
github.api_url           # https://api.github.com
github.graphql_url       # https://api.github.com/graphql
github.action            # action ID for the current step
github.action_path       # path where action is located (composite/JS actions)
github.action_ref        # ref of the action (e.g., v4)
github.action_repository # owner/repo of the action
github.workspace         # default working directory on runner
github.base_ref          # PR base branch (only on pull_request)
github.head_ref          # PR head branch (only on pull_request)
github.event.number      # PR/issue number
github.event.pull_request.head.sha  # actual head commit of PR
github.event.pull_request.merged    # boolean, true after merge
```

### Other Contexts

```yaml
# env: environment variables set in workflow/job/step
env.MY_VAR

# vars: repository/environment/organization variables (non-secret)
vars.DEPLOYMENT_URL

# secrets: encrypted secrets — masked in logs
secrets.GITHUB_TOKEN
secrets.DEPLOY_KEY

# needs: outputs from dependent jobs
needs.build.outputs.version
needs.build.result              # success, failure, cancelled, skipped

# strategy/matrix: current matrix values
strategy.fail-fast              # boolean
strategy.max-parallel           # number
strategy.job-index              # 0-based index in matrix
strategy.job-total              # total matrix combinations
matrix.os                       # current matrix value

# steps: outputs/outcome from previous steps in same job
steps.my_step.outputs.result
steps.my_step.outcome           # success, failure, cancelled, skipped
steps.my_step.conclusion        # same as outcome after continue-on-error

# job: job-level info
job.container.id                # container ID if job uses container
job.container.network           # container network
job.services.<service>.id       # service container ID
job.status                      # current job status

# runner: runner machine details
runner.name                     # runner name
runner.os                       # Linux, Windows, macOS
runner.arch                     # X86, X64, ARM, ARM64
runner.temp                     # temp directory path
runner.tool_cache               # tool cache directory
runner.debug                    # '1' if debug logging enabled
runner.environment              # github-hosted or self-hosted

# inputs: workflow_dispatch or workflow_call inputs
inputs.environment
inputs.version
```

## Context Availability Matrix

Not all contexts are available everywhere:

| Context | `env:` (workflow) | `jobs.<id>.if` | `jobs.<id>.steps.if` | `jobs.<id>.steps.run` | `jobs.<id>.outputs` |
|---------|:-:|:-:|:-:|:-:|:-:|
| `github` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `env` | partial | ✗ | ✓ | ✓ | ✓ |
| `vars` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `secrets` | ✗ | ✗ | ✓ | ✓ | ✗ |
| `needs` | ✗ | ✓ | ✓ | ✓ | ✓ |
| `strategy` | ✗ | ✗ | ✓ | ✓ | ✗ |
| `matrix` | ✗ | ✗ | ✓ | ✓ | ✗ |
| `steps` | ✗ | ✗ | ✓ | ✓ | ✓ |
| `inputs` | ✓ | ✓ | ✓ | ✓ | ✓ |
| `runner` | ✗ | ✗ | ✓ | ✓ | ✗ |

**Critical**: `secrets` is NOT available in `jobs.<id>.if` — you cannot conditionally skip a job based on whether a secret exists by comparing it directly. Use a prior job to set an output flag instead.

## Functions

### String Functions

```yaml
contains('Hello World', 'World')          # true
contains(github.event.issue.labels.*.name, 'bug')  # array search
startsWith('refs/heads/main', 'refs/heads')         # true
endsWith(github.ref, '/main')                       # true
format('Hello {0}, run #{1}', github.actor, github.run_number)
```

### Data Functions

```yaml
toJSON(github.event)              # serialize to JSON string
fromJSON(steps.meta.outputs.json) # parse JSON string to object
fromJSON('true')                  # convert string to boolean (for inputs)
join(github.event.issue.labels.*.name, ', ')  # array to string
hashFiles('**/package-lock.json') # SHA-256 hash for caching
hashFiles('**/*.gradle', '**/gradle.properties')  # multiple patterns
```

### Status Check Functions

```yaml
success()     # true if all prior steps succeeded (default implicit condition)
failure()     # true if any prior step failed
always()      # always runs, even if cancelled — use for cleanup
cancelled()   # true if workflow was cancelled
```

**Combining status checks**:
```yaml
- if: always() && steps.test.outcome == 'failure'  # run on failure only, even if cancelled
- if: ${{ !cancelled() }}   # run unless explicitly cancelled (succeeds on failure too)
- if: success() || failure()  # run on success or failure, but NOT if cancelled
```

## Operators and Type Coercion

### Operators
```
==  !=  <  >  <=  >=      Comparison (case-insensitive for strings!)
&&  ||  !                 Logical
( )                        Grouping
.                          Property dereference
[ ]                        Index access
*                          Object filter (property projection)
```

### Type Coercion Rules
| Type | Null | Boolean | Number | String |
|------|------|---------|--------|--------|
| Null | — | `false` | `0` | `''` |
| Boolean | — | — | `true→1, false→0` | `'true'/'false'` |
| Number | — | `0→false, else→true` | — | string of number |
| String | — | `''→false, else→true` | parsed or NaN | — |

**String comparison is CASE-INSENSITIVE**: `'ABC' == 'abc'` is `true`.

## Object Filter Syntax

Project a property from an array of objects:

```yaml
# Get all label names from an issue
github.event.issue.labels.*.name
# Result: ['bug', 'enhancement', 'priority-high']

# Use with contains() for label checks
if: contains(github.event.issue.labels.*.name, 'deploy')
```

## Expression Gotchas

```yaml
# Boolean workflow_dispatch inputs are STRINGS — must cast
- if: inputs.dry_run == 'true'          # string comparison works
- if: fromJSON(inputs.dry_run) == true  # proper boolean comparison

# hashFiles returns empty string '' when no files match — not null
- if: hashFiles('**/pnpm-lock.yaml') != ''

# Ternary-style conditional
env:
  TAG: ${{ github.ref_type == 'tag' && github.ref_name || 'latest' }}

# Multiline expressions (use > or | YAML scalars)
- if: >-
    github.event_name == 'push' &&
    startsWith(github.ref, 'refs/tags/v')
```
