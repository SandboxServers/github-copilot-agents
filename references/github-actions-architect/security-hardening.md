# Security Hardening for GitHub Actions — Deep Reference

## Permissions: Principle of Least Privilege

Always start with empty permissions at the workflow level and grant per-job.

```yaml
# Workflow-level: deny all by default
permissions: {}

jobs:
  lint:
    runs-on: ubuntu-latest
    permissions:
      contents: read        # checkout only
    steps:
      - uses: actions/checkout@v4

  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write       # OIDC for cloud auth
      packages: write       # push container images
    steps:
      - uses: actions/checkout@v4
```

### All Available Permission Scopes

| Scope | Values | Controls |
|-------|--------|----------|
| `actions` | read / write / none | Workflow runs, artifacts, caches |
| `contents` | read / write / none | Repository contents, commits, branches, tags |
| `deployments` | read / write / none | Deployment environments and statuses |
| `id-token` | write / none | OIDC token requests (no read — only write or none) |
| `issues` | read / write / none | Issues and comments |
| `packages` | read / write / none | GitHub Packages (GHCR, npm, NuGet, etc.) |
| `pages` | read / write / none | GitHub Pages site deployments |
| `pull-requests` | read / write / none | PR comments, reviews, merge |
| `security-events` | read / write / none | Code scanning alerts, Dependabot |
| `statuses` | read / write / none | Commit status checks |
| `attestations` | read / write / none | Artifact attestations |
| `checks` | read / write / none | Check runs and check suites |
| `discussions` | read / write / none | Repository discussions |
| `repository-projects` | read / write / none | Project boards |

**Key rule**: Job-level permissions can only *restrict* from the workflow-level default — they cannot escalate. If the workflow grants `contents: read`, a job cannot set `contents: write`.

## SHA Pinning for Third-Party Actions

Always pin third-party actions to full 40-character commit SHAs. Comment the version for readability.

```yaml
steps:
  # GOOD — pinned to SHA with version comment
  - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
  - uses: actions/setup-node@49933ea5288caeca8642d1e84afbd3f7d6820020 # v4.4.0
  - uses: docker/build-push-action@1dc73863535b631f98b2378be8619f6b6f2ac52e # v6.12.0

  # BAD — tag can be moved by attacker
  - uses: actions/checkout@v4
  # BAD — mutable branch reference
  - uses: some-org/some-action@main
```

**Dependabot auto-updates SHA pins**. Add to `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
    # Dependabot will submit PRs updating SHA pins with version comments
```

## Script Injection Prevention

**NEVER** interpolate `${{ }}` expressions directly in `run:` blocks. An attacker who controls the interpolated value can inject arbitrary shell commands.

### Attacker-Controlled Contexts (Full List)

All of these can contain malicious payloads and must NEVER appear in `run:` scripts:

- `github.event.pull_request.title`
- `github.event.pull_request.body`
- `github.event.pull_request.head.ref` (branch name)
- `github.event.pull_request.head.label`
- `github.event.comment.body`
- `github.event.review.body`
- `github.event.review_comment.body`
- `github.event.issue.title`
- `github.event.issue.body`
- `github.event.pages.*.page_name`
- `github.event.commits.*.message`
- `github.event.commits.*.author.name`
- `github.event.commits.*.author.email`
- `github.event.discussion.title`
- `github.event.discussion.body`
- `github.head_ref` (PR source branch)

### Safe Pattern: Environment Variable Intermediary

```yaml
# DANGEROUS — direct interpolation enables injection
- name: Bad example
  run: echo "Title is ${{ github.event.pull_request.title }}"

# SAFE — env var intermediary prevents injection
- name: Good example
  env:
    PR_TITLE: ${{ github.event.pull_request.title }}
  run: |
    echo "Title is $PR_TITLE"
    if [[ "$PR_TITLE" =~ ^feat ]]; then
      echo "Feature PR detected"
    fi
```

## Fork and Pull Request Security

### `pull_request` (safe for forks)
- Runs in the context of the **merge commit** (fork code merged into base)
- Gets a **read-only** `GITHUB_TOKEN` scoped to the fork
- **Cannot access** repository secrets
- Safe for untrusted code from external contributors

### `pull_request_target` (dangerous — use with extreme caution)
- Runs in the context of the **base branch** (your repository code)
- Gets a **read-write** `GITHUB_TOKEN` for your repository
- **Has access** to all repository secrets
- The PR code is NOT checked out by default — `actions/checkout` gets the base branch

```yaml
# SAFE pull_request_target pattern — metadata only, never checkout PR code
on: pull_request_target
jobs:
  label:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      # Only reads event metadata — never checks out or runs PR code
      - uses: actions/labeler@v5

# DANGEROUS — checking out PR code with pull_request_target gives
# attacker code access to your secrets
on: pull_request_target
jobs:
  build:
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}  # NEVER DO THIS
      - run: npm install  # Attacker's package.json runs with your secrets!
```

## Secret Management

### Secret Hierarchy and Precedence
1. **Environment secrets** — scoped to a deployment environment, highest priority
2. **Repository secrets** — available to all workflows in a repo
3. **Organization secrets** — shared across repos, can be scoped to specific repos or visibility

Environment secrets **override** repository secrets of the same name.

### Masking Derived Values

Secrets are auto-masked, but derived values are NOT. Use `::add-mask::`:

```yaml
- name: Derive and mask token
  run: |
    DERIVED_TOKEN=$(echo "${{ secrets.KEY }}" | base64)
    echo "::add-mask::$DERIVED_TOKEN"
    # Now $DERIVED_TOKEN will be redacted in logs
    echo "Using token: $DERIVED_TOKEN"
```

**Never store structured data (JSON, XML) as a single secret** — redaction relies on exact string matching and structured data often fails to redact properly. Create individual secrets for each value.

## GitHub App Tokens

When `GITHUB_TOKEN` lacks the needed permissions (e.g., triggering workflows, cross-repo access):

```yaml
steps:
  - uses: actions/create-github-app-token@v2
    id: app-token
    with:
      app-id: ${{ vars.APP_ID }}
      private-key: ${{ secrets.APP_PRIVATE_KEY }}
      repositories: "repo-a,repo-b"  # optional: scope to specific repos

  - uses: actions/checkout@v4
    with:
      token: ${{ steps.app-token.outputs.token }}
      # This token CAN trigger further workflow runs (unlike GITHUB_TOKEN)
```

## CODEOWNERS for Workflow Protection

Protect `.github/workflows/` by requiring review from a security or platform team:

```text
# .github/CODEOWNERS
.github/workflows/  @my-org/platform-team
.github/actions/    @my-org/platform-team
```

Combine with branch protection rules requiring CODEOWNERS approval before merge.

## Supply Chain Security

### Artifact Attestations

```yaml
- uses: actions/attest-build-provenance@v2
  with:
    subject-path: 'dist/my-app'
    # Generates SLSA provenance attestation signed with Sigstore
```

### Dependency Review on PRs

```yaml
- uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: moderate
    deny-licenses: GPL-3.0, AGPL-3.0
    # Blocks PRs that introduce vulnerable or license-restricted dependencies
```

### OpenSSF Scorecards

```yaml
- uses: ossf/scorecard-action@v2
  with:
    results_file: results.sarif
    results_format: sarif
    publish_results: true
    # Uploads to GitHub code scanning for visibility
```

## Organization-Level Policies

| Policy | Setting |
|--------|---------|
| Require SHA pinning | Actions settings → require full SHA for third-party actions |
| Restrict allowed actions | Only allow actions from verified creators or specific repos |
| Disable fork PR workflows | Prevent workflows from running on fork PRs (for private repos) |
| Prevent Actions from approving PRs | Disallow workflow-created PR approvals |

## Token Exposure Mitigation

If a secret is accidentally logged:
1. **Immediately rotate** the exposed secret
2. **Delete the workflow run logs** (Settings → Actions → Run logs)
3. **Audit usage** — check if the token was used outside expected contexts
4. **Review and re-mask** — ensure derived values use `::add-mask::`
5. Consider using **environment protection rules** with required reviewers for high-value secrets
