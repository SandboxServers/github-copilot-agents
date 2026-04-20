# Repository & Source Control Migration: ADO → GitHub

## Git Repo Migration (ADO Git → GitHub Git)

### Method 1: Mirror Clone (Recommended for Most Cases)

Preserves all branches, tags, and full commit history:

```bash
# Clone the bare repository from ADO
git clone --mirror https://dev.azure.com/{org}/{project}/_git/{repo}
cd {repo}.git

# Add GitHub as a new remote
git remote add github https://github.com/{org}/{repo}.git

# Push everything to GitHub
git push github --mirror
```

**Important:** `--mirror` pushes all refs (branches, tags, notes). Review before pushing —
remove any refs you don't want to carry over (e.g., ADO internal refs).

### Method 2: GitHub CLI

```bash
# Create the GitHub repo first
gh repo create {org}/{repo} --private --source=. --push

# Or create empty and push separately
gh repo create {org}/{repo} --private
git remote add github https://github.com/{org}/{repo}.git
git push github --all
git push github --tags
```

### Method 3: GitHub Import (Web UI)

GitHub's web-based import (github.com/new/import) can import from any Git URL including ADO.
Suitable for simple repos but less control over the process.

### Method 4: Azure DevOps Migrator / gh actions-importer

`gh actions-importer` focuses on pipeline migration but the general approach includes:
1. Audit source repos and pipelines
2. Create target repos
3. Convert pipelines during migration

### Large Repository Considerations

| Issue | Solution |
|---|---|
| Repo > 1 GB | Clean history with `git filter-repo`, remove large binaries |
| Large binary files | Migrate to Git LFS before pushing to GitHub |
| Deep history | Consider shallow clone for working copies post-migration |
| GitHub push limit (2 GB) | Split into multiple pushes or contact GitHub support |

```bash
# Migrate large files to LFS before push
git lfs migrate import --include="*.zip,*.dll,*.exe,*.nupkg" --everything
git push github --mirror
```

## TFVC to Git Conversion

TFVC (Team Foundation Version Control) requires conversion to Git before GitHub migration.

### git-tfs (Primary Tool)

`git-tfs` is the established tool for TFVC → Git conversion:

```bash
# Install git-tfs (Windows)
choco install gittfs

# Clone TFVC repo to local Git (full history)
git tfs clone https://dev.azure.com/{org} $/Project/Main --branches=all

# Clone with limited history (faster, recommended for large repos)
git tfs clone https://dev.azure.com/{org} $/Project/Main --from=changeset-number

# Quick clone (latest only, no history)
git tfs quick-clone https://dev.azure.com/{org} $/Project/Main
```

### TFVC Branch Mapping

TFVC uses folder-based branching ($/Project/Main, $/Project/Dev, $/Project/Release).
Map these to Git branches during conversion:

```
TFVC Path                    → Git Branch
$/Project/Main               → main
$/Project/Development        → develop
$/Project/Release/v1.0       → release/v1.0
$/Project/Feature/work-item  → feature/work-item
```

### Selective History Migration

Full TFVC history conversion can take hours/days for large repos. Consider:
- **Full history:** Required for compliance/audit trails
- **Partial history:** `--from=changeset` to start from a specific point
- **Tip only:** `quick-clone` for fastest migration when history isn't needed

### Large Binary Files in TFVC

TFVC commonly stores binaries (DLLs, NuGet packages, images). During conversion:
1. Identify binaries: `git lfs migrate info --everything`
2. Convert to LFS: `git lfs migrate import --include="*.dll,*.exe" --everything`
3. Set up `.gitattributes` for ongoing LFS tracking
4. Consider moving binaries to artifact feeds (NuGet, npm) instead of repo storage

## Multi-Repo Considerations

### ADO Multi-Repo Checkout → GHA

ADO uses `resources.repositories` for multi-repo pipelines:

```yaml
# ADO pattern
resources:
  repositories:
    - repository: templates
      type: git
      name: ProjectName/TemplateRepo
      ref: main

steps:
  - checkout: self
  - checkout: templates
```

```yaml
# GHA equivalent — multiple actions/checkout steps
steps:
  - uses: actions/checkout@v4                    # Primary repo

  - uses: actions/checkout@v4
    with:
      repository: org/template-repo              # Secondary repo
      path: templates                            # Checkout to subdirectory
      token: ${{ secrets.REPO_ACCESS_TOKEN }}    # PAT or GitHub App token for private repos
```

### Mono-Repo vs Multi-Repo Evaluation

Migration is a natural inflection point to re-evaluate repo structure:

| Factor | Mono-Repo | Multi-Repo |
|---|---|---|
| Shared code changes | Atomic commits across services | PRs to shared libraries |
| CI complexity | Path filters per service | Simpler per-repo workflows |
| Access control | GitHub CODEOWNERS per path | Repo-level permissions |
| Dependency management | Direct references | Package feeds / submodules |
| GitHub Actions fit | path-filter triggers | Natural repo separation |

### Submodule Strategies

If ADO repos reference each other, evaluate:
- **Git submodules** — complex but maintains separate repos
- **Package feeds** — publish shared code as packages (NuGet, npm)
- **Monorepo consolidation** — merge related repos during migration

## Branch Policy Migration

### ADO Branch Policies → GitHub Branch Protection Rules + Rulesets

| ADO Branch Policy | GitHub Equivalent |
|---|---|
| Require minimum reviewers | Branch protection: required reviews |
| Check for linked work items | Required status check (custom action) |
| Check for comment resolution | Require conversation resolution |
| Build validation | Required status checks |
| Merge strategy (squash only, etc.) | Allowed merge types in repo settings |
| Path-based policies | Rulesets with path-based targeting |
| Automatically included reviewers | CODEOWNERS file + required review from code owners |
| Reset votes on new pushes | Dismiss stale pull request approvals |

### GitHub Rulesets (Recommended over Branch Protection)

Rulesets are the modern replacement, offering:
- **Organization-wide rules** — apply across all repos from one config
- **Layered rules** — multiple rulesets can apply to the same branch
- **Tag protection** — protect tag patterns (not possible with branch protection)
- **Bypass lists** — specific actors can bypass (emergency access)
- **Enforcement modes** — active, evaluate (audit mode), disabled

```
Repository Settings → Rules → Rulesets → New ruleset
- Target: branches matching "main", "release/*"
- Rules: require PR, require 2 approvals, require status checks,
         require conversation resolution, block force pushes
- Bypass: repository-admin role only
```

## Code Review Migration

| ADO Feature | GitHub Equivalent |
|---|---|
| Required reviewers | CODEOWNERS + required review from code owners |
| Optional reviewers | PR review requests (no enforcement) |
| Auto-complete | Auto-merge (enable in repo settings) |
| Draft PRs | Draft PRs (direct equivalent) |
| PR templates | `.github/pull_request_template.md` |
| PR work item linking | Issue references in PR body (`Closes #123`) |
| Merge strategies | Repo settings: merge commit, squash, rebase |
| Branch lock | Branch protection / rulesets |

## Wiki Migration

### ADO Wiki → GitHub

ADO wikis are Git repos. Migration options:

1. **GitHub Wiki** — clone ADO wiki repo, push to GitHub wiki repo (`{repo}.wiki.git`)
2. **docs/ folder** — move wiki content into `docs/` directory in the main repo
3. **GitHub Pages** — publish docs via GitHub Pages with MkDocs, Docusaurus, etc.

```bash
# Clone ADO wiki (it's a Git repo)
git clone https://dev.azure.com/{org}/{project}/_git/{project}.wiki

# Push to GitHub wiki
cd {project}.wiki
git remote add github https://github.com/{org}/{repo}.wiki.git
git push github --mirror
```

**Note:** ADO wiki uses `.md` files with ADO-specific syntax (e.g., `::: mermaid`, `[[_TOC_]]`).
Convert these to standard Markdown or GitHub-flavored Markdown during migration.

## Work Item Linking

ADO commits reference work items with `AB#1234` syntax. After migration:
- GitHub uses `#123` to reference issues
- If work items are not migrated to GitHub Issues, links become non-functional
- Consider: migrate work items to GitHub Issues, or maintain cross-references via URLs
- ADO Boards has a GitHub integration that can bridge the gap during transition
