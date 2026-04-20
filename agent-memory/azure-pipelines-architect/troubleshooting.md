# Azure Pipelines Troubleshooting Guide

## YAML Parsing and Template Errors

### Indentation Issues

YAML is whitespace-sensitive. Mixed tabs/spaces or wrong indentation depth causes silent misparses or hard failures.

```yaml
# WRONG — steps indented under wrong parent
jobs:
  - job: Build
  steps:                      # Error: 'steps' is a sibling of job, not a child
    - script: echo hello

# CORRECT
jobs:
  - job: Build
    steps:                    # Properly nested under the job
      - script: echo hello
```

### Template Expression Evaluation Failures

```yaml
# ERROR: "Unexpected value ''"  — variable is empty at compile time
variables:
  env: ''
stages:
  - stage: Deploy
    condition: ${{ eq(variables.env, 'prod') }}    # Compile-time — may not have value yet

# FIX: Use runtime expression for variables that aren't set until runtime
    condition: eq(variables['env'], 'prod')          # Runtime evaluation
```

**Key rule:** `${{ }}` evaluates at **compile time** (template expansion). `$[ ]` and bare expressions in `condition:` evaluate at **runtime**. Using compile-time syntax for runtime values is the #1 source of expression bugs.

### Template Parameter Type Mismatches

```yaml
# Template expects a boolean, caller passes a string
# template.yml
parameters:
  - name: deploy
    type: boolean

# caller.yml — ERROR: "Expected Boolean, got String"
extends:
  template: template.yml
  parameters:
    deploy: 'true'        # String, not boolean

# FIX:
    deploy: true           # Boolean literal (no quotes)
```

## Variable Expansion Failures

### Wrong Syntax for Context

| Syntax | Context | Evaluated | Example |
|--------|---------|-----------|---------|
| `$(var)` | Macro | Before task runs | `$(Build.SourceBranch)` |
| `${{ variables.var }}` | Template | Compile time | `${{ variables.env }}` |
| `$[variables.var]` | Expression | Runtime | `$[variables.dynamicVal]` |
| `$(var)` in script | Macro expansion | Substituted by agent | `echo $(myVar)` |
| `$env:VAR` / `$VAR` | Shell | Shell environment | `$env:BUILD_SOURCEBRANCH` (PS) |

```yaml
# PROBLEM: Script variable not expanding
steps:
  - bash: echo $MYVAR              # Empty! Macro syntax not used
    env:
      MYVAR: $(myPipelineVar)      # FIX: Map pipeline var to env var

# PROBLEM: Secret not available in template expression
variables:
  - group: my-secrets
steps:
  - script: echo ${{ variables.mySecret }}   # EMPTY — secrets blocked at compile time
  - script: echo $(mySecret)                  # WORKS — macro expansion at runtime (masked)
```

## Trigger Issues

### Pipeline Didn't Trigger

Common causes and fixes:

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Push to `main` doesn't trigger | Path filters exclude all changed files | Check `paths.include`/`exclude` |
| PR doesn't trigger | PR trigger not configured or `drafts: false` | Add `pr:` section to YAML |
| Pipeline runs on wrong branch | Trigger includes wildcard matching unintended branches | Narrow branch filter |
| Fork PR doesn't build | Fork builds disabled by default | Enable in pipeline settings (security risk) |
| Batch mode skips commits | `batch: true` coalesces — by design | Remove `batch: true` if every commit needs a build |
| Scheduled trigger missed | Schedule uses UTC, not local time | Verify cron in UTC timezone |

```yaml
# Debug: verify trigger evaluation
# Check pipeline run "Trigger" tab — shows why the run was (or wasn't) started
# REST API: GET /_apis/pipelines/{id}/runs?api-version=7.1 — see triggerInfo
```

## Agent Issues

### Agent Disconnected

- Check agent service status: `systemctl status vsts.agent.*.service` (Linux) or Services panel (Windows)
- Verify network connectivity to `https://dev.azure.com`
- Check PAT expiration — agents using expired PATs disconnect silently
- Review agent diagnostic logs: `_diag/Agent_*.log` in the agent directory

### Demands Not Met

```
Error: No agent found in pool 'MyPool' which satisfies the specified demands: maven, docker

# Fix: Check demands vs capabilities
# Pipeline demands (from pool.demands + task demands)
# Agent capabilities (auto-detected + user-defined)
# View: Agent pools > Agents > [agent] > Capabilities
```

## Service Connection Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `AADSTS7000215: Invalid client secret` | Service principal secret expired | Rotate secret in Azure AD, update service connection |
| `could not be accessed` | Pipeline not authorized | Authorize pipeline on the service connection |
| `Insufficient privileges` | SP missing required Azure RBAC role | Assign correct role (e.g., Contributor) to SP |
| `TF400813: Resource not available` | Service connection deleted or renamed | Recreate or update reference |

## Environment and Approval Issues

- **Stuck deployment**: Approval pending — check environment approvals in pipeline run UI
- **Approval timeout**: Default is 30 days; configure shorter timeout in check settings
- **Exclusive lock**: Another run holds the lock — wait or cancel the blocking run
- **Business hours check failure**: Deployment attempted outside configured window

## Cache Miss Debugging

```yaml
# Enable system.debug to see cache key evaluation
variables:
  system.debug: true

# Cache log output shows:
# - Resolved cache key segments
# - Hash values for file-based segments
# - Whether a cache hit, inexact hit, or miss occurred
# - RestoreKeys attempted and their match results
```

Cache key evaluation order:
1. Exact match on full key → `cacheHitVar = true`
2. Prefix match on `restoreKeys` (tried in order) → `cacheHitVar = inexact`
3. No match → `cacheHitVar = false`, post-job saves new cache

## Debugging Techniques

### Enable Verbose Logs

```yaml
# Method 1: Pipeline variable
variables:
  system.debug: true

# Method 2: Queue-time variable
# When running manually, add variable: system.debug = true

# Method 3: Specific to a run (REST API)
# POST /_apis/pipelines/{id}/runs with body: { "variables": { "system.debug": { "value": "true" } } }
```

### REST API Inspection

```bash
# Get pipeline run details including triggered reason and variables
GET https://dev.azure.com/{org}/{project}/_apis/pipelines/{pipelineId}/runs/{runId}?api-version=7.1

# Get timeline — shows every task's status, duration, and issues
GET https://dev.azure.com/{org}/{project}/_apis/build/builds/{buildId}/timeline?api-version=7.1

# Preview a pipeline run (validate YAML without running)
POST https://dev.azure.com/{org}/{project}/_apis/pipelines/{pipelineId}/runs?api-version=7.1
Body: { "previewRun": true }
```

### Common Error Message Reference

| Error Message | Meaning | Resolution |
|---------------|---------|------------|
| `An error occurred while loading the YAML build pipeline` | YAML syntax error | Validate with YAML linter; check for tabs |
| `Template expression is not valid` | Invalid `${{ }}` expression | Check function names, parameter types |
| `Variable group could not be found` | Missing or no access to variable group | Verify group exists and pipeline is authorized |
| `No hosted parallelism has been purchased or granted` | Free tier exhausted | Purchase parallel jobs or request free grant |
| `The pipeline is not valid. Job X: Stage Y depends on unknown stage Z` | Mismatched `dependsOn` | Check stage/job names for typos |
| `Could not find a pool with name X` | Pool doesn't exist or no access | Check pool name spelling and project permissions |
| `##[error]Script failed with error: ...` | Inline script error | Check script syntax, exit codes, error handling |
