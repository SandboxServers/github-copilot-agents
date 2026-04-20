# GitHub Actions Workflow Triggers — Deep Reference

## push and pull_request Triggers

### Branch and Tag Filters

```yaml
on:
  push:
    branches:
      - main
      - 'releases/**'        # glob: matches releases/v1, releases/v1/hotfix
      - '!releases/**-alpha'  # negation: excludes alpha branches
    tags:
      - 'v[0-9]+.[0-9]+.[0-9]+'  # semver tags only
    paths:
      - 'src/**'
      - '!src/**/*.test.ts'       # ignore test files
```

**Mutually exclusive filters**: `branches` and `branches-ignore` cannot be used together. Same for `tags`/`tags-ignore` and `paths`/`paths-ignore`. Use negation patterns (`!`) within `branches` instead.

```yaml
# WRONG — will error
on:
  push:
    branches: [main]
    branches-ignore: [feature/**]

# CORRECT — use negation
on:
  push:
    branches:
      - main
      - 'feature/**'
      - '!feature/experimental-*'
```

**Path filter behavior**: When all changed files match `paths-ignore` (or none match `paths`), the workflow won't run. If both `paths` and branch filters exist, both must match.

## pull_request vs pull_request_target — CRITICAL SECURITY

```yaml
# SAFE: runs fork code with read-only GITHUB_TOKEN, no secrets
on:
  pull_request:
    types: [opened, synchronize, reopened]

# DANGEROUS if misused: runs BASE branch code with write token + secrets
on:
  pull_request_target:
    types: [opened, synchronize]
```

**Security rules for `pull_request_target`**:
- Code from the BASE branch runs, NOT the PR head
- Has full write GITHUB_TOKEN and access to secrets
- **NEVER** do `actions/checkout@v4` with `ref: ${{ github.event.pull_request.head.sha }}` and then execute that code — this gives untrusted PR code access to your secrets
- Safe use: labeling PRs, posting comments, reading PR metadata without executing PR code

```yaml
# SAFE pull_request_target: only labels, never checks out PR code
on:
  pull_request_target:
    types: [opened]
jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/labeler@v5
```

## Activity Types

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened, closed, labeled, unlabeled,
            ready_for_review, converted_to_draft, review_requested,
            review_request_removed, assigned, unassigned, edited,
            auto_merge_enabled, auto_merge_disabled, milestoned, demilestoned]
  issues:
    types: [opened, edited, deleted, transferred, pinned, unpinned,
            closed, reopened, assigned, unassigned, labeled, unlabeled,
            locked, unlocked, milestoned, demilestoned]
  release:
    types: [published, unpublished, created, edited, deleted, prereleased, released]
```

**Default types if omitted**: `pull_request` defaults to `[opened, synchronize, reopened]`. `issues` defaults to `[opened, edited, deleted]`.

## workflow_dispatch — Manual Triggers with Typed Inputs

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options: [dev, staging, production]
        default: dev
      dry_run:
        description: 'Dry run mode'
        required: false
        type: boolean
        default: true
      version:
        description: 'Version to deploy'
        required: true
        type: string
      concurrency_limit:
        description: 'Max parallel deploys'
        type: number
        default: 3
      target_env:
        description: 'GitHub environment to use'
        type: environment

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.target_env || inputs.environment }}
    steps:
      - run: echo "Deploying ${{ inputs.version }} (dry_run=${{ inputs.dry_run }})"
```

**Gotcha**: Boolean inputs arrive as strings from the API. Use `fromJSON()` to cast: `if: fromJSON(inputs.dry_run) == false`.

## workflow_call — Reusable Workflow Trigger

```yaml
on:
  workflow_call:
    inputs:
      artifact_name:
        type: string
        required: true
    outputs:
      build_version:
        description: 'The computed build version'
        value: ${{ jobs.build.outputs.version }}
    secrets:
      DEPLOY_KEY:
        required: true
```

See `reusable-workflows.md` for full coverage.

## workflow_run — Chaining Workflows

```yaml
on:
  workflow_run:
    workflows: ["Build"]          # name of triggering workflow
    types: [completed]            # completed, requested, in_progress
    branches: [main]              # optional branch filter

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Build succeeded on ${{ github.event.workflow_run.head_branch }}"
```

**Key details**: The triggered workflow runs on the DEFAULT branch's version of the workflow file, not the branch that triggered the upstream workflow. Use `github.event.workflow_run.head_branch` to know the source.

## schedule — Cron Triggers

```yaml
on:
  schedule:
    - cron: '30 5 * * 1-5'   # 5:30 UTC, weekdays only
    - cron: '0 0 * * 0'      # midnight UTC, Sundays
```

**Caveats**: Always UTC. Runs only on default branch. Minimum interval 5 minutes. May be delayed up to 30+ minutes under heavy GitHub load. Disabled automatically after 60 days of no repo activity.

## repository_dispatch — API-Triggered Workflows

```yaml
on:
  repository_dispatch:
    types: [deploy-production, run-integration-tests]

jobs:
  handle:
    runs-on: ubuntu-latest
    if: github.event.action == 'deploy-production'
    steps:
      - run: echo "Payload version: ${{ github.event.client_payload.version }}"
```

Trigger via API:
```bash
curl -X POST -H "Authorization: token $TOKEN" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/OWNER/REPO/dispatches \
  -d '{"event_type":"deploy-production","client_payload":{"version":"2.1.0"}}'
```

## merge_group — Merge Queue Integration

```yaml
on:
  merge_group:
    types: [checks_requested]
    branches: [main]
```

Runs against the temporary merge group branch created by GitHub's merge queue. Use alongside `pull_request` to validate both PR checks and merge queue checks.

## Common Trigger Issues — Why Didn't My Workflow Run?

| Symptom | Cause |
|---------|-------|
| Push to branch doesn't trigger | Branch filter doesn't match; check glob pattern |
| PR from fork doesn't run | `pull_request_target` required for secrets; `pull_request` won't get secrets from forks |
| Schedule never fires | Repo inactive >60 days; workflow disabled; only runs on default branch |
| `paths` filter skipped job | All changed files excluded by paths filter |
| Activity type ignored | Event defaults don't include your type; explicitly list types |
| `workflow_run` not triggered | Workflow name must match exactly (case-sensitive) |
| Workflow not visible | File must be on default branch for `schedule`, `repository_dispatch` |

## Event-Specific Context Payloads

Each trigger populates `github.event` differently:
- `push`: `github.event.commits[]`, `github.event.head_commit`, `github.event.before`, `github.event.after`
- `pull_request`: `github.event.pull_request.number`, `.head.sha`, `.base.ref`, `.merged`, `.draft`
- `workflow_dispatch`: `github.event.inputs.<name>`
- `schedule`: `github.event.schedule` (the cron string that matched)
- `workflow_run`: `github.event.workflow_run.conclusion`, `.head_branch`, `.id`
