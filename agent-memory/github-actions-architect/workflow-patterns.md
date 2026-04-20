# Common Production Workflow Patterns — Deep Reference

## CI Pattern: Lint → Test → Build

```yaml
name: CI
on:
  pull_request:
    branches: [main]
    paths-ignore:
      - 'docs/**'
      - '*.md'
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    needs: lint
    strategy:
      matrix:
        node: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: 'npm'
      - run: npm ci
      - run: npm test

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: ./dist/
```

## Monorepo Pattern: Path-Based Detection

```yaml
name: Monorepo CI
on:
  pull_request:
    branches: [main]

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      api: ${{ steps.filter.outputs.api }}
      web: ${{ steps.filter.outputs.web }}
      shared: ${{ steps.filter.outputs.shared }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            api:
              - 'services/api/**'
              - 'packages/shared/**'
            web:
              - 'apps/web/**'
              - 'packages/shared/**'
            shared:
              - 'packages/shared/**'

  api-ci:
    needs: detect-changes
    if: needs.detect-changes.outputs.api == 'true'
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: services/api
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test

  web-ci:
    needs: detect-changes
    if: needs.detect-changes.outputs.web == 'true'
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: apps/web
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
```

## Release Pattern: Tag Push → Build → Publish → GitHub Release

```yaml
name: Release
on:
  push:
    tags:
      - 'v[0-9]+.[0-9]+.[0-9]+'

permissions:
  contents: write
  packages: write
  attestations: write
  id-token: write

jobs:
  build-and-release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
          registry-url: 'https://npm.pkg.github.com'

      - run: npm ci
      - run: npm run build
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - uses: actions/attest-build-provenance@v2
        with:
          subject-path: './dist/**'

      - uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true
          files: |
            dist/*.tgz
            dist/*.zip
```

## Deployment Pattern: Progressive Environments

```yaml
name: Deploy
on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: app
          path: ./dist/

  deploy-dev:
    needs: build
    runs-on: ubuntu-latest
    environment: development
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app
      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
      - run: az webapp deploy --resource-group rg-dev --name app-dev --src-path ./dist/

  deploy-staging:
    needs: deploy-dev
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app
      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
      - run: az webapp deploy --resource-group rg-staging --name app-staging --src-path ./dist/

  deploy-prod:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://www.example.com
    # Environment protection rules enforce manual approval before this job runs
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app
      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
      - run: az webapp deploy --resource-group rg-prod --name app-prod --src-path ./dist/
```

## PR Pattern: Status Checks + Auto-Merge

```yaml
name: PR Checks
on:
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

concurrency:
  group: pr-${{ github.event.pull_request.number }}
  cancel-in-progress: true

permissions:
  contents: read
  pull-requests: write

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test && npm run build

  label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/labeler@v5
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}

  # Enable merge queue for serialized merges to main
  # Configure in repo settings: require 'ci' job to pass
```

## Scheduled Maintenance Pattern

```yaml
name: Scheduled Maintenance
on:
  schedule:
    - cron: '0 6 * * 1'   # Every Monday at 6 AM UTC

permissions:
  issues: write
  pull-requests: write

jobs:
  stale-issues:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/stale@v9
        with:
          stale-issue-message: 'This issue has been inactive for 60 days.'
          days-before-stale: 60
          days-before-close: 14

  health-check:
    runs-on: ubuntu-latest
    steps:
      - run: |
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://api.example.com/health)
          if [ "$STATUS" != "200" ]; then
            echo "Health check failed: $STATUS"
            exit 1
          fi
```

## Notification Pattern: Failure Alerts

```yaml
name: Notify on Failure
on:
  workflow_run:
    workflows: ["Deploy"]
    types: [completed]

jobs:
  notify:
    if: github.event.workflow_run.conclusion == 'failure'
    runs-on: ubuntu-latest
    steps:
      - name: Send Slack notification
        uses: slackapi/slack-github-action@v2
        with:
          webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
          webhook-type: incoming-webhook
          payload: |
            {
              "text": "Deployment failed on ${{ github.event.workflow_run.head_branch }}",
              "blocks": [{
                "type": "section",
                "text": {
                  "type": "mrkdwn",
                  "text": "*${{ github.event.workflow_run.name }}* failed\nBranch: `${{ github.event.workflow_run.head_branch }}`\n<${{ github.event.workflow_run.html_url }}|View Run>"
                }
              }]
            }
```

## Container Build Pattern: Build → Scan → Push with Attestation

```yaml
name: Container Build
on:
  push:
    branches: [main]
    paths:
      - 'Dockerfile'
      - 'src/**'

permissions:
  contents: read
  packages: write
  attestations: write
  id-token: write

jobs:
  build-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: docker/metadata-action@v5
        id: meta
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=sha
            type=ref,event=branch
            type=semver,pattern={{version}}

      - uses: docker/build-push-action@v6
        id: push
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - uses: actions/attest-build-provenance@v2
        with:
          subject-name: ghcr.io/${{ github.repository }}
          subject-digest: ${{ steps.push.outputs.digest }}
          push-to-registry: true
```

## Multi-Platform Build Pattern

```yaml
jobs:
  build:
    strategy:
      matrix:
        include:
          - os: ubuntu-latest
            target: linux-x64
          - os: ubuntu-latest-arm64
            target: linux-arm64
          - os: windows-latest
            target: win-x64
          - os: macos-latest
            target: osx-arm64
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - run: dotnet publish -r ${{ matrix.target }} -o ./out/
      - uses: actions/upload-artifact@v4
        with:
          name: app-${{ matrix.target }}
          path: ./out/

  combine:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          path: ./all-platforms/
          pattern: app-*
```

## Rollback Pattern: Manual Redeployment

```yaml
name: Rollback
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version tag to rollback to (e.g., v1.2.3)'
        required: true
        type: string
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options:
          - staging
          - production

permissions:
  id-token: write
  contents: read

jobs:
  rollback:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ inputs.version }}

      - run: npm ci && npm run build

      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}

      - run: |
          SLOT="rollback-$(date +%s)"
          az webapp deployment slot create --name app-${{ inputs.environment }} \
            --resource-group rg-${{ inputs.environment }} --slot $SLOT
          az webapp deploy --name app-${{ inputs.environment }} \
            --resource-group rg-${{ inputs.environment }} --slot $SLOT --src-path ./dist/
          az webapp deployment slot swap --name app-${{ inputs.environment }} \
            --resource-group rg-${{ inputs.environment }} --slot $SLOT --target-slot production
```

## Reusable CI Pattern: Shared Across Repos

```yaml
# .github/workflows/ci-node.yml in the shared-workflows repo
name: Reusable Node CI
on:
  workflow_call:
    inputs:
      node_version:
        type: string
        default: '20'
      working_directory:
        type: string
        default: '.'

jobs:
  ci:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ${{ inputs.working_directory }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node_version }}
          cache: 'npm'
          cache-dependency-path: '${{ inputs.working_directory }}/package-lock.json'
      - run: npm ci
      - run: npm run lint --if-present
      - run: npm test
      - run: npm run build --if-present
```

Called from consuming repos:

```yaml
# In any repo's .github/workflows/ci.yml
name: CI
on: [pull_request, push]
jobs:
  ci:
    uses: my-org/shared-workflows/.github/workflows/ci-node.yml@main
    with:
      node_version: '20'
```
