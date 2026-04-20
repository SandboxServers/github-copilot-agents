# Reusable Workflows (workflow_call) — Deep Reference

## Defining a Reusable Workflow

A reusable workflow declares `on: workflow_call` with typed inputs, outputs, and secrets.

```yaml
# .github/workflows/build-and-test.yml (reusable)
name: Build and Test
on:
  workflow_call:
    inputs:
      node_version:
        description: 'Node.js version to use'
        type: string
        required: false
        default: '20'
      run_e2e:
        description: 'Whether to run E2E tests'
        type: boolean
        required: false
        default: false
      artifact_retention:
        description: 'Days to retain artifacts'
        type: number
        required: false
        default: 5
    outputs:
      build_sha:
        description: 'Git SHA of the build'
        value: ${{ jobs.build.outputs.sha }}
      test_passed:
        description: 'Whether all tests passed'
        value: ${{ jobs.test.outputs.passed }}
    secrets:
      NPM_TOKEN:
        description: 'npm registry auth token'
        required: true
      SONAR_TOKEN:
        required: false

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      sha: ${{ steps.meta.outputs.sha }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node_version }}
      - run: npm ci
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
      - run: npm run build
      - id: meta
        run: echo "sha=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

  test:
    needs: build
    runs-on: ubuntu-latest
    outputs:
      passed: ${{ steps.result.outputs.passed }}
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
      - if: inputs.run_e2e
        run: npm run test:e2e
      - id: result
        if: always()
        run: echo "passed=${{ job.status == 'success' }}" >> $GITHUB_OUTPUT
```

## Calling a Reusable Workflow

```yaml
# .github/workflows/ci.yml (caller)
name: CI Pipeline
on:
  push:
    branches: [main]
  pull_request:

jobs:
  build-and-test:
    uses: ./.github/workflows/build-and-test.yml          # same repo
    with:
      node_version: '20'
      run_e2e: ${{ github.event_name == 'push' }}
      artifact_retention: 7
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  # Cross-repo call — must specify ref
  shared-lint:
    uses: my-org/shared-workflows/.github/workflows/lint.yml@v2
    with:
      config_path: '.eslintrc.json'
    secrets: inherit    # pass ALL caller's secrets

  deploy:
    needs: [build-and-test, shared-lint]
    if: github.ref == 'refs/heads/main'
    uses: ./.github/workflows/deploy.yml
    with:
      build_sha: ${{ needs.build-and-test.outputs.build_sha }}
    secrets: inherit
```

## Output Passing Chain

The full chain for getting data from a reusable workflow step to the caller:

```
1. Step writes to $GITHUB_OUTPUT          →  steps.<id>.outputs.<name>
2. Job declares in jobs.<id>.outputs      →  jobs.<id>.outputs.<name>
3. Workflow maps in on.workflow_call.outputs  →  caller reads via needs.<job>.outputs.<name>
```

```yaml
# Inside reusable workflow — step output
- id: version
  run: echo "tag=v1.2.3" >> $GITHUB_OUTPUT

# Inside reusable workflow — job output (required bridge)
jobs:
  compute:
    outputs:
      tag: ${{ steps.version.outputs.tag }}

# Inside reusable workflow — workflow output
on:
  workflow_call:
    outputs:
      release_tag:
        value: ${{ jobs.compute.outputs.tag }}

# In caller workflow — consume output
jobs:
  call-build:
    uses: ./.github/workflows/compute.yml
  publish:
    needs: call-build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Tag is ${{ needs.call-build.outputs.release_tag }}"
```

## Nesting Rules and Limits

- **Max depth**: 10 levels of reusable workflow nesting (A calls B calls C... max 10)
- **No loops**: Workflow A cannot call workflow B if B calls A (directly or transitively)
- **Ref requirement**: Cross-repo calls MUST specify a ref (`@main`, `@v2`, `@sha`)
- **Same-repo calls**: Use `./.github/workflows/file.yml` — always uses the SAME commit as the caller
- **Private repos**: Reusable workflows can only be called from the SAME organization

## Permissions — Only Reduce, Never Elevate

```yaml
# Caller has:     contents: write, issues: write
# Reusable sets:  contents: read
# Effective:      contents: read  (reduced by reusable)

# Reusable CANNOT grant permissions the caller doesn't have
# Reusable CANNOT escalate beyond what the caller's GITHUB_TOKEN provides
```

If the reusable workflow doesn't declare `permissions:`, it inherits the caller's permissions. If it DOES declare `permissions:`, it gets only what it declares (additive to the caller's grants, but capped at the caller's level).

## strategy + uses Mutual Exclusion

A job CANNOT have both `strategy.matrix` and `uses` (reusable workflow call). To run a reusable workflow with matrix, put the matrix on the CALLER:

```yaml
# WRONG — will error
jobs:
  deploy:
    strategy:
      matrix:
        env: [dev, staging, prod]
    uses: ./.github/workflows/deploy.yml  # cannot combine

# CORRECT — matrix on the caller job
jobs:
  deploy:
    strategy:
      matrix:
        target: [dev, staging, prod]
    uses: ./.github/workflows/deploy.yml
    with:
      environment: ${{ matrix.target }}
    secrets: inherit
```

## Secret Propagation Rules

- `secrets: inherit` passes ALL of the caller's secrets to the reusable workflow
- Named secrets (`secrets: { KEY: ${{ secrets.KEY }} }`) pass only specified secrets
- In a chain (A→B→C): C receives secrets from B only, not directly from A (unless B also forwards them)
- **Environment secrets**: A caller CANNOT pass environment-scoped secrets to a reusable workflow. The reusable workflow must declare its own `environment:` to access environment secrets.
- Secrets in reusable workflows are masked in logs just like regular secrets

```yaml
# Reusable workflow accessing environment secrets directly
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production    # this job's steps get production secrets
    steps:
      - run: deploy --key ${{ secrets.PROD_API_KEY }}
```

## Design Patterns

### Shared CI Workflow
One reusable workflow for build+test, parameterized by language version and test type. All repos call it identically.

### Shared Deployment Workflow
Standard deployment reusable workflow accepting environment name, image tag, and region. Enforces deployment gates and approval workflows.

### Parameterized Testing
Reusable test workflow accepting test suite name, parallelism count, and coverage threshold.

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Over-nesting (5+ levels) | Hard to debug, slow to load | Flatten; use composite actions for shared steps |
| Monolithic reusable workflow | Too many inputs, rigid | Split into focused workflows (build, test, deploy) |
| Secrets leaking via outputs | Outputting secret values | Never map secrets to workflow outputs |
| Missing required secrets | Runtime failure in reusable workflow | Use `required: true` and document in README |
| Hardcoded refs for same-repo | Breaks on branches | Use `./` relative path for same-repo calls |
| Passing secrets individually when all are needed | Verbose, easy to miss new secrets | Use `secrets: inherit` |
