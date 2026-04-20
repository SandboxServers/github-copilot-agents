# Caching and Artifact Management — Deep Reference

## actions/cache@v4 — Dependency Caching

### Key Design Pattern: Most Specific → Least Specific

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
      ${{ runner.os }}-
```

Matching order:
1. **Exact match** on `key` → cache hit (no new cache saved)
2. **Prefix match** on `key` → cache restored, new cache saved with full key at job end
3. **Sequential** `restore-keys` prefix matching → most recently created cache wins
4. **No match** → cold start, new cache saved at job end

### hashFiles() for Cache Keys

```yaml
# Hash a single lock file
key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

# Hash multiple files (useful for .NET with multiple csproj)
key: ${{ runner.os }}-dotnet-${{ hashFiles('**/*.csproj', '**/Directory.Packages.props') }}

# Hash all go.sum files in a monorepo
key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
```

**Important**: `hashFiles()` returns an **empty string** if no files match the glob pattern. This creates a cache key without a hash suffix, causing false cache hits. Always verify your globs match.

## Cache Scope and Visibility

| Workflow trigger | Cache readable from |
|-----------------|-------------------|
| Branch push | Same branch + default branch |
| Pull request | PR branch + base branch + default branch |
| Tag push | Default branch only |

- PRs can **read** the base branch cache (warm start on first PR build)
- PRs create their own cache entries scoped to `refs/pull/<number>/merge`
- Feature branches **cannot** read caches from other feature branches

## Cache Eviction Rules

- **10 GB limit** per repository (all branches combined)
- Entries untouched for **7 days** are evicted
- When the limit is hit, **LRU (Least Recently Used)** entries are evicted first
- No manual cache deletion via actions (use REST API or GitHub CLI: `gh cache delete`)

## Language-Specific Caching with setup-* Actions

These actions have **built-in** cache support — no separate `actions/cache` step needed:

```yaml
# Node.js — caches ~/.npm or node_modules based on lock file
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: 'npm'              # also supports 'yarn' and 'pnpm'

# Python — caches pip download directory
- uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: 'pip'              # also supports 'pipenv' and 'poetry'

# Java — caches Maven or Gradle local repository
- uses: actions/setup-java@v4
  with:
    distribution: 'temurin'
    java-version: '21'
    cache: 'maven'            # also supports 'gradle' and 'sbt'

# Go — caches module download and build cache
- uses: actions/setup-go@v5
  with:
    go-version: '1.22'
    cache: true               # boolean — caches ~/go/pkg/mod and ~/.cache/go-build
```

### .NET Caching (Manual — no setup-dotnet cache option)

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.nuget/packages
    key: ${{ runner.os }}-nuget-${{ hashFiles('**/*.csproj', '**/Directory.Packages.props') }}
    restore-keys: |
      ${{ runner.os }}-nuget-

- uses: actions/setup-dotnet@v4
  with:
    dotnet-version: '9.0.x'
```

## Save-Only and Restore-Only

For fine-grained control, use the separate save and restore actions:

```yaml
# Restore at the start of the job — don't fail if cache miss
- uses: actions/cache/restore@v4
  id: cache-restore
  with:
    path: ./build-output
    key: build-${{ github.sha }}
    restore-keys: build-

# ... do expensive build work ...

# Save only if content changed (e.g., new build)
- uses: actions/cache/save@v4
  if: steps.cache-restore.outputs.cache-hit != 'true'
  with:
    path: ./build-output
    key: build-${{ github.sha }}
```

Use cases:
- **Restore-only**: Jobs that consume caches but shouldn't create new ones (e.g., deployment)
- **Save-only**: Jobs that build caches for downstream consumers
- **Conditional save**: Avoid overwriting a cache that was already a hit

## Artifact Management (actions/upload-artifact@v4 + actions/download-artifact@v4)

### v4+ Breaking Changes (New Backend)
- Artifacts are **immutable** — you cannot overwrite an artifact with the same name in the same run
- Each upload creates a **separate** artifact (no more appending to the same name)
- `actions/download-artifact@v4` **only** downloads v4 artifacts (not v3)
- Artifacts are **not shared across workflow runs** — only between jobs in the same run

### Sharing Between Jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: dotnet publish -o ./publish
      - uses: actions/upload-artifact@v4
        with:
          name: app-build
          path: ./publish/
          retention-days: 5

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app-build
          path: ./deploy-package/
      - run: az webapp deploy --src-path ./deploy-package/

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app-build
          path: ./test-package/
      - run: dotnet test ./test-package/
```

### Artifact Retention

| Plan | Default Retention | Maximum |
|------|------------------|---------|
| GitHub Free | 90 days | 90 days |
| GitHub Pro | 90 days | 90 days |
| GitHub Team | 90 days | 90 days |
| GitHub Enterprise | 90 days | 400 days |

Configure per-upload:

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: ./results/
    retention-days: 7    # 1-90 (or up to 400 for Enterprise)
```

### Download All Artifacts

```yaml
# Download all artifacts from the current workflow run
- uses: actions/download-artifact@v4
  with:
    path: ./all-artifacts/
    # Each artifact extracted to its own subdirectory: ./all-artifacts/<name>/
```

### Merge Multiple Outputs into One Artifact

```yaml
# Matrix job outputs — each uploads with a unique name
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
steps:
  - uses: actions/upload-artifact@v4
    with:
      name: build-${{ matrix.os }}
      path: ./dist/

# Later job — merge artifacts
steps:
  - uses: actions/upload-artifact/merge@v4
    with:
      name: all-builds
      pattern: build-*
```

## Large Artifacts — Consider Cloud Storage

For artifacts over ~500 MB, upload directly to cloud storage with OIDC:

```yaml
# Upload to Azure Blob Storage instead of GitHub Artifacts
- uses: azure/login@v2
  with:
    client-id: ${{ vars.AZURE_CLIENT_ID }}
    tenant-id: ${{ vars.AZURE_TENANT_ID }}
    subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}

- run: |
    az storage blob upload-batch \
      --source ./large-output \
      --destination artifacts \
      --account-name mystorageaccount \
      --auth-mode login \
      --overwrite
```

## Cache vs Artifact — When to Use Each

| Use Case | Cache | Artifact |
|----------|-------|----------|
| Dependencies (npm, NuGet, pip) | Yes | No |
| Build output shared between jobs | No | Yes |
| Persists across workflow runs | Yes (7-day LRU) | No (single run only) |
| Downloadable from UI | No | Yes |
| Immutable per workflow run | No (key-based) | Yes (v4+) |
| Size limit | 10 GB per repo | ~10 GB per run |

## Performance Tips

- **Cache dependencies, not build output** — caches persist across runs; artifacts don't
- **Use `hashFiles()` on lock files** — ensures cache invalidation when deps change
- **Minimize upload size** — use `.gitignore`-aware globs, exclude `node_modules/.cache`
- **Prefer setup-* built-in caching** — simpler configuration, handles restore-keys automatically
- **Set short retention** for non-essential artifacts — reduces storage consumption
- **Use `if-no-files-found: error`** on uploads to catch broken build paths early:

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: app-build
    path: ./dist/
    if-no-files-found: error   # 'warn' (default) or 'ignore'
```
