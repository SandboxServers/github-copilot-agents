# Matrix Strategies for GitHub Actions — Deep Reference

## Basic Matrix — Single and Multi-Dimension

```yaml
# Single dimension: 3 jobs
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - run: echo "Testing on ${{ matrix.os }}"

# Multi-dimension cross-product: 3 × 3 = 9 jobs
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: ['18', '20', '22']
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm test
```

## include — Adding and Expanding Combinations

`include` adds entries to the matrix. It can expand existing combinations with extra variables, or add entirely new combinations.

```yaml
jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node: ['18', '20']
        include:
          # Add a variable to a specific existing combination
          - os: ubuntu-latest
            node: '20'
            coverage: true          # only this combo gets coverage

          # Add an entirely new combination not in the cross-product
          - os: macos-latest
            node: '20'
            experimental: true

          # Introduce a new variable for ALL include entries matching
          - os: windows-latest
            shell: pwsh
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm test
      - if: matrix.coverage
        run: npm run coverage
```

**include matching rules**: If an `include` entry matches ALL existing key-value pairs of a cross-product entry, the extra variables are merged into that entry. If it doesn't match any existing entry, it's added as a new combination.

## exclude — Removing Combinations

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: ['18', '20', '22']
        exclude:
          # Remove Node 18 on Windows (known incompatibility)
          - os: windows-latest
            node: '18'
          # Remove Node 22 on macOS (not yet supported)
          - os: macos-latest
            node: '22'
    # Results: 9 - 2 = 7 jobs
```

**Processing order**: `exclude` is applied first, then `include` adds entries. You cannot exclude an `include`-only entry.

## Dynamic Matrices with fromJSON

Generate matrix values from a prior job's output:

```yaml
jobs:
  prepare:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - id: set-matrix
        run: |
          # Generate matrix from changed directories, API response, config file, etc.
          MATRIX=$(cat matrix-config.json)
          echo "matrix=$MATRIX" >> $GITHUB_OUTPUT

      # Or build dynamically
      - id: set-matrix-dynamic
        run: |
          ENVS='["dev","staging"]'
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            ENVS='["dev","staging","production"]'
          fi
          echo "matrix={\"environment\":$ENVS}" >> $GITHUB_OUTPUT

  deploy:
    needs: prepare
    strategy:
      matrix: ${{ fromJSON(needs.prepare.outputs.matrix) }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to ${{ matrix.environment }}"
```

**fromJSON gotcha**: The output must be a valid JSON string. Use `toJSON()` for debugging: `echo "${{ toJSON(matrix) }}"`.

## fail-fast and continue-on-error

```yaml
jobs:
  test:
    strategy:
      fail-fast: true       # DEFAULT: cancel all matrix jobs if one fails
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - run: npm test

  experimental:
    strategy:
      fail-fast: false      # let all jobs finish regardless of failures
      matrix:
        node: ['20', '22', '23-nightly']
    runs-on: ubuntu-latest
    continue-on-error: ${{ matrix.node == '23-nightly' }}  # per-leg error tolerance
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm test
```

**`continue-on-error` vs `fail-fast`**: `fail-fast: false` lets all jobs run but still marks the workflow as failed. `continue-on-error: true` on a job makes that job's failure not affect the workflow conclusion.

## max-parallel — Throttling Concurrent Jobs

```yaml
jobs:
  deploy:
    strategy:
      max-parallel: 2       # only 2 deployments at a time
      fail-fast: false
      matrix:
        region: [us-east-1, us-west-2, eu-west-1, ap-southeast-1]
    runs-on: ubuntu-latest
    steps:
      - run: deploy --region ${{ matrix.region }}
```

Use when deploying to rate-limited APIs, databases, or shared infrastructure.

## Matrix with Reusable Workflows

The `strategy` goes on the CALLER job, not inside the reusable workflow:

```yaml
jobs:
  deploy:
    strategy:
      matrix:
        environment: [dev, staging, prod]
      max-parallel: 1                           # deploy sequentially
      fail-fast: true
    uses: ./.github/workflows/deploy.yml
    with:
      target_env: ${{ matrix.environment }}
    secrets: inherit
```

## Conditional Matrix Legs

```yaml
jobs:
  build:
    strategy:
      matrix:
        include:
          - target: linux
            os: ubuntu-latest
            deploy: true
          - target: windows
            os: windows-latest
            deploy: false
          - target: macos
            os: macos-latest
            deploy: ${{ github.ref == 'refs/heads/main' }}
    runs-on: ${{ matrix.os }}
    steps:
      - run: make build
      - if: matrix.deploy
        run: make deploy
```

## Matrix Limits

| Limit | Value |
|-------|-------|
| Max jobs per workflow run | 256 |
| Max matrix entries per dimension | 256 (across all dimensions, capped at 256 total combinations) |
| Max workflow run time | 35 days |
| Max concurrent jobs (free tier) | 20 |
| Max concurrent macOS jobs (free tier) | 5 |

## Complete Examples

### CI Test Matrix

```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest]
        python: ['3.10', '3.11', '3.12']
        include:
          - os: ubuntu-latest
            python: '3.12'
            coverage: true
        exclude:
          - os: windows-latest
            python: '3.10'
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python }}
      - run: pip install -e ".[test]"
      - run: pytest
      - if: matrix.coverage
        run: pytest --cov --cov-report=xml
      - if: matrix.coverage
        uses: codecov/codecov-action@v4
```

### Deployment Matrix with Sequential Execution

```yaml
name: Deploy
on:
  workflow_dispatch:
    inputs:
      version:
        required: true
        type: string
jobs:
  deploy:
    strategy:
      max-parallel: 1
      fail-fast: true
      matrix:
        include:
          - env: staging
            url: https://staging.example.com
          - env: production
            url: https://example.com
    runs-on: ubuntu-latest
    environment:
      name: ${{ matrix.env }}
      url: ${{ matrix.url }}
    steps:
      - run: deploy --env ${{ matrix.env }} --version ${{ inputs.version }}
      - run: smoke-test --url ${{ matrix.url }}
```
