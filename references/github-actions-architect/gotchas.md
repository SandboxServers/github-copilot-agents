# GitHub Actions Gotchas — Deep Reference

## Expression Context Availability

Not all contexts are available everywhere. `secrets` is unavailable in `if:` conditions at the workflow level. `needs` context is only available in jobs that declare `needs:`. The `env` context from workflow-level is not available in `jobs.<id>.if`.

| Context | Availability |
|---------|-------------|
| `github` | Everywhere |
| `env` | Steps only (workflow `env` not in job-level `if`) |
| `secrets` | Steps and reusable workflow inputs |
| `needs` | Only in jobs with `needs:` declared |
| `matrix` | Only inside the job using `strategy.matrix` |

## Matrix Strategy: failFast Default Is True

By default, `fail-fast: true` — if any matrix job fails, all other in-progress matrix jobs are cancelled immediately. This can hide failures in other combinations.

```yaml
strategy:
  fail-fast: false  # Let all matrix jobs complete
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: [18, 20, 22]
```

## Reusable Workflow Nesting Limit

Reusable workflows can only be nested **4 levels deep**. Workflow A calls B calls C calls D — that's the maximum. Level 5 fails with a cryptic error. Also, reusable workflows cannot call other reusable workflows in a loop.

## Secrets Not Available in Fork PRs

For `pull_request` events from forked repositories, `secrets` (except `GITHUB_TOKEN`) are **empty** by default. This is a deliberate security measure. Use `pull_request_target` with caution — it runs in the context of the base branch with full secrets access but the PR code is untrusted.

**Workaround:** Use `pull_request_target` with explicit checkout of the PR HEAD only for read-only operations. Never run untrusted code with elevated permissions.

## Concurrency Group Cancellation Risks

Setting `cancel-in-progress: true` on deployment workflows can leave infrastructure in a partially deployed state. The cancelled job does not run cleanup steps unless you use `if: cancelled()`.

```yaml
concurrency:
  group: deploy-production
  cancel-in-progress: false  # Safer for deployments

# For CI builds, cancellation is usually fine:
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

## Composite Actions Cannot Access Secrets

Composite actions do not have access to the `secrets` context. Secrets must be passed as explicit inputs from the calling workflow.

```yaml
# In composite action.yml — secrets must come via inputs
inputs:
  api-key:
    description: 'API key passed from calling workflow'
    required: true
```

## Environment Protection Rules Block Entire Jobs

Required reviewers on environments block the **entire job**, not just the deployment step. If a job has a long build phase before the deploy step, the reviewer must approve before any of it runs.

**Fix:** Split build and deploy into separate jobs. The deploy job references the environment and waits for approval.

## GITHUB_TOKEN Permission Differences

Default `GITHUB_TOKEN` permissions differ by trigger:
- **push/pull_request (same repo):** Read for contents, write for some scopes (depends on org settings)
- **pull_request from forks:** Read-only across all scopes
- **Org setting "Read repository contents and packages permissions":** Can restrict default to read-only globally

Always declare explicit `permissions:` blocks. Never rely on defaults.

## Job Outputs Must Be Explicitly Declared

Step outputs (`$GITHUB_OUTPUT`) don't automatically propagate to job outputs. You must explicitly map them.

```yaml
jobs:
  build:
    outputs:
      version: ${{ steps.ver.outputs.version }}  # Must declare here
    steps:
      - id: ver
        run: echo "version=1.2.3" >> "$GITHUB_OUTPUT"
  deploy:
    needs: build
    steps:
      - run: echo "${{ needs.build.outputs.version }}"
```

## Artifact Size Limits

- Maximum artifact size: **10 GB** per artifact
- Recommended for performance: under **500 MB**
- Total storage limit varies by plan (500 MB free, 2 GB Pro, 50 GB Enterprise)
- Large artifacts significantly slow upload/download and can cause timeouts
- Artifacts are immutable once uploaded — same name in same run overwrites

## Schedule Trigger Limitations

- Minimum interval: **5 minutes** (but GitHub may throttle)
- Schedules only run on the **default branch**
- GitHub may **delay or skip** scheduled runs during high load periods
- Scheduled workflows on inactive repos (no commits in 60 days) are **automatically disabled**
- Cron uses **UTC timezone only**

```yaml
on:
  schedule:
    - cron: '0 */6 * * *'  # Every 6 hours — reliable
    # Avoid '* * * * *' — will be throttled
```
