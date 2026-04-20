# Composite Actions, JavaScript Actions, and Docker Actions — Deep Reference

## Composite Actions (using: composite)

Composite actions bundle multiple steps into a single reusable action. They run directly in the caller's runner environment.

### action.yml — Complete Example

```yaml
# .github/actions/setup-and-build/action.yml
name: 'Setup and Build'
description: 'Install dependencies and build the project'
branding:
  icon: 'package'
  color: 'blue'

inputs:
  node_version:
    description: 'Node.js version'
    required: false
    default: '20'
  build_args:
    description: 'Extra arguments for build command'
    required: false
    default: ''
  npm_token:
    description: 'NPM auth token (pass secrets as inputs!)'
    required: false

outputs:
  artifact_path:
    description: 'Path to build output'
    value: ${{ steps.build.outputs.dist_path }}
  cache_hit:
    description: 'Whether cache was restored'
    value: ${{ steps.cache.outputs.cache-hit }}

runs:
  using: 'composite'
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node_version }}

    - name: Cache dependencies
      id: cache
      uses: actions/cache@v4
      with:
        path: node_modules
        key: deps-${{ runner.os }}-${{ hashFiles('package-lock.json') }}

    - name: Install dependencies
      if: steps.cache.outputs.cache-hit != 'true'
      shell: bash                    # REQUIRED on every run step
      run: |
        npm ci
      env:
        NODE_AUTH_TOKEN: ${{ inputs.npm_token }}

    - name: Build
      id: build
      shell: bash                    # REQUIRED — not inherited from caller
      run: |
        npm run build ${{ inputs.build_args }}
        echo "dist_path=dist/" >> $GITHUB_OUTPUT

    - name: Read action's own files
      shell: bash
      run: |
        # github.action_path = directory where THIS action lives
        cat "${{ github.action_path }}/config/defaults.json"
```

### Critical Composite Action Rules

**Shell is REQUIRED per step**: Every `run:` step must declare `shell:`. It is NOT inherited from the caller workflow's `defaults.run.shell`. This is the #1 composite action pitfall.

**Secrets are NOT directly available**: Composite actions cannot access `secrets.*` context. Callers must pass secrets as inputs:

```yaml
# In caller workflow
- uses: ./.github/actions/setup-and-build
  with:
    npm_token: ${{ secrets.NPM_TOKEN }}    # secret passed as input

# In composite action — use inputs.npm_token, NOT secrets.NPM_TOKEN
```

**Workspace sharing**: Composite actions run in the caller's `$GITHUB_WORKSPACE`. They share the same working directory. Use `github.action_path` to reference files bundled with the action itself.

**Outputs via $GITHUB_OUTPUT**: Steps set outputs using `echo "key=value" >> $GITHUB_OUTPUT`, then map them in the action's `outputs:` section referencing `steps.<id>.outputs.<key>`.

**Conditional steps**: `if:` conditions work in composite action steps, using the same expression syntax as workflows.

## JavaScript Actions (using: node20)

Use JavaScript actions when you need complex logic, API calls, or access to the GitHub Actions toolkit.

```yaml
# action.yml for a JavaScript action
name: 'PR Comment'
description: 'Post a formatted comment on a pull request'
inputs:
  message:
    description: 'Comment body'
    required: true
  pr_number:
    description: 'PR number'
    required: true
outputs:
  comment_id:
    description: 'ID of the created comment'
runs:
  using: 'node20'
  main: 'dist/index.js'       # main entry point
  pre: 'dist/setup.js'        # runs before main (optional)
  post: 'dist/cleanup.js'     # runs after job completes (optional)
  post-if: 'always()'         # condition for post step
```

### Key Toolkit Packages

```javascript
const core = require('@actions/core');       // inputs, outputs, logging, secrets
const github = require('@actions/github');   // Octokit client, context
const exec = require('@actions/exec');       // run shell commands
const io = require('@actions/io');           // file operations
const cache = require('@actions/cache');     // cache save/restore
const artifact = require('@actions/artifact'); // upload/download artifacts
const glob = require('@actions/glob');       // file globbing
const toolCache = require('@actions/tool-cache'); // download and cache tools

// Usage example
const token = core.getInput('github_token', { required: true });
const octokit = github.getOctokit(token);
core.setOutput('comment_id', result.data.id);
core.setSecret(token);  // mask in logs
```

## Docker Container Actions (using: docker)

Docker actions run in an isolated container. Linux runners only.

```yaml
# action.yml for a Docker action
name: 'Security Scan'
description: 'Run security scanning tool'
inputs:
  scan_path:
    description: 'Path to scan'
    default: '.'
  severity:
    description: 'Minimum severity'
    default: 'HIGH'
runs:
  using: 'docker'
  image: 'Dockerfile'           # build from Dockerfile in action repo
  # image: 'docker://ghcr.io/org/scanner:v2'  # or use pre-built image
  args:
    - ${{ inputs.scan_path }}
    - '--severity'
    - ${{ inputs.severity }}
  entrypoint: '/scanner'        # override Dockerfile ENTRYPOINT (optional)
  env:
    SCAN_CONFIG: '/etc/scanner/config.yml'
  pre-entrypoint: 'setup.sh'   # runs before main entrypoint (optional)
  post-entrypoint: 'cleanup.sh' # runs after main entrypoint (optional)
```

**Docker action trade-offs**: Slower startup (image pull/build), but fully isolated environment with exact dependency control. Only runs on Linux runners.

## Action Versioning — Best Practices

```yaml
# BEST: SHA pinning — immutable, tamper-proof
- uses: actions/checkout@a5ac7e51b41094c92402da3b24376905380afc29  # v4.1.6

# GOOD: Exact version tag
- uses: actions/checkout@v4.1.6

# ACCEPTABLE: Major version tag (author must maintain)
- uses: actions/checkout@v4

# AVOID: Branch reference — mutable, can break without notice
- uses: actions/checkout@main
```

**Dependabot for actions**: Auto-updates action versions in workflows.

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
    groups:
      actions:
        patterns: ["*"]
```

## Design Patterns

### Wrapping Complex Shell Logic
Extract multi-step shell sequences (install tool, configure, run, parse output) into a composite action with clean inputs/outputs.

### Shared Setup
Common setup steps (checkout, language setup, cache restore, authentication) bundled into a single action call per project type.

### Environment Preparation
Composite action that configures cloud credentials, sets environment variables, and validates prerequisites before deployment steps.

## Testing Composite Actions

- **Local testing**: Use [nektos/act](https://github.com/nektos/act) to run workflows locally
- **Integration tests**: Create a test workflow in the action repo that exercises all input combinations
- **Version matrix**: Test against multiple runner OS versions when the action should be cross-platform

```yaml
# .github/workflows/test-action.yml
name: Test Composite Action
on: [push, pull_request]
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: ./                     # test the action from this repo
        with:
          node_version: '20'
      - run: test -d dist/           # verify expected output
        shell: bash
```
