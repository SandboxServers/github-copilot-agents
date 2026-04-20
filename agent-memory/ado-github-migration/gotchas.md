# ADO → GitHub Migration Gotchas

## Large Repos with Binary Files

Git history containing large binaries (NuGet packages, compiled assets, database backups) causes migration timeouts and inflated clone sizes. GitHub has a 100 MB file limit and a 5 GB recommended repo size.

**Mitigation:** Use `git filter-repo` to strip binaries before migration. Move large files to Git LFS. For TFVC repos with binary history, consider a tip-only migration (latest code, no history). Run `git rev-list --objects --all | git cat-file --batch-check | sort -k3 -n` to identify the largest objects before attempting migration.

## Service Connections to OIDC: Different Trust Model

ADO service connections authenticate with service principals using client secrets or certificates. GitHub OIDC uses federated credentials with subject claims based on repo, branch, and environment. The trust boundary is completely different.

**Impact:** You cannot just copy credentials. You must create new federated credentials in Entra ID with subject claims matching the GitHub repository/environment. Each deployment target needs its own federated identity configuration.

## Variable Groups Have No Direct Equivalent

ADO variable groups bundle related variables (including Key Vault references) into a reusable unit linked to multiple pipelines. GitHub has no grouping mechanism — only individual secrets and variables at org, repo, or environment scope.

**Impact:** Plan variable-to-scope mapping carefully. Key Vault-linked variable groups require a Key Vault read step in the workflow instead of native integration. See `variable-group-migration.md` for detailed patterns.

## ADO Environments vs GitHub Environments

ADO environments support approval gates, exclusive locks, VM-targeted deployments, and Kubernetes resource targeting. GitHub environments support required reviewers, wait timers, branch filters, and deployment protection rules — but no VM or Kubernetes targeting.

**Impact:** VM-based deployments need a different approach (self-hosted runners or SSH steps). Kubernetes deployments use `kubeconfig` secrets rather than environment-level resources. Approval workflows work differently — GitHub uses required reviewers, not sequential approval chains.

## Task Groups vs Composite Actions

ADO task groups support parameters with default values, conditional task insertion, and nested task groups. GitHub composite actions support inputs with defaults but have no conditional step logic within the action itself and no nesting.

**Impact:** Complex task groups with conditional logic must be split into multiple composite actions or converted to reusable workflows. Parameter models differ — ADO uses `${{ parameters.x }}` while GitHub composite actions use `${{ inputs.x }}`.

## Branch Policies vs Branch Protection Rules

ADO branch policies support minimum reviewer count, required specific reviewers, build validation (specific pipelines), comment resolution, merge strategy, work item linking, and automatic reviewers based on paths. GitHub branch protection rules (and rulesets) support similar but not identical features.

**Key differences:** GitHub rulesets can target multiple branches with patterns. ADO's "build validation" maps to required status checks. ADO's auto-reviewer by path maps to CODEOWNERS. ADO's "reset votes on push" has no direct equivalent — GitHub always resets approvals on new pushes (configurable via "dismiss stale reviews").

## Classic Release Pipelines Have No YAML

ADO classic release pipelines (the visual designer) have no YAML representation. They cannot be exported to YAML within ADO, and `gh actions-importer` has limited support for converting them.

**Impact:** Classic releases must be manually reconstructed as GHA workflows. Document the stage sequence, gates, variable scopes, and artifact sources before building the equivalent workflow.

## Work Item Links in Commits

ADO auto-links work items referenced in commit messages (`#1234` or `AB#1234`). After migration to GitHub, these references point nowhere — GitHub uses `#` for issues/PRs in the *current* repo.

**Impact:** Remap or document the link format. Consider adding a migration note to repos explaining the old reference format. For ongoing traceability, use GitHub Issues or link back to read-only ADO work items.

## ADO Wikis: No Direct Migration Path

ADO project wikis are stored in a special Git repo (`ProjectName.wiki`). GitHub wikis are also Git-backed but have different page naming conventions, no nested folder support in the UI, and different markdown extensions.

**Impact:** Clone the ADO wiki repo, restructure file names (replace spaces, flatten hierarchy), and push to the GitHub wiki repo. Mermaid diagrams and ADO-specific markdown (`::: moniker`) need manual conversion.

## Build Numbers and Retention

ADO auto-generates build numbers with configurable format (`$(Date:yyyyMMdd).$(Rev:r)`) and has per-pipeline retention policies with keep-forever flags. GitHub Actions uses sequential `run_number` and `run_attempt` with a fixed 400-day log retention.

**Impact:** If your processes depend on custom build number formats (for artifact naming, traceability), replicate the format using workflow expressions. Retention requires external artifact storage for long-term needs.

## ADO Test Plans: No GitHub Native Equivalent

ADO Test Plans (test suites, test cases, test runs with pass/fail tracking, manual testing UI) have no counterpart in GitHub. Test *result publishing* (from automated tests) maps to third-party actions, but manual test management does not.

**Impact:** Choose a third-party tool or process for manual test management. Automated test results can be published using actions like `dorny/test-reporter` or `mikepenz/action-junit-report`.

## ADO Artifacts vs GitHub Packages

ADO Artifacts supports NuGet, npm, Maven, Python, and Universal Packages with upstream sources and per-feed permissions. GitHub Packages supports the same feed types but scopes permissions differently (repo-level, not feed-level) and has different rate limits.

**Impact:** Upstream source behavior differs. ADO's Universal Packages have no GitHub equivalent — use GitHub Releases or Azure Blob Storage. Feed-level permissions in ADO must be mapped to repository or organization-level permissions in GitHub.

## Scheduled Trigger Syntax Differences

ADO scheduled triggers use `schedules:` with `cron:` and support branch inclusion/exclusion filters. GitHub Actions uses `schedule:` with `cron:` but only runs on the default branch. Feature-branch scheduled runs are not supported.

**Impact:** If you rely on scheduled builds for non-default branches in ADO, you must restructure the workflow. Use `workflow_dispatch` with a branch input or run the schedule on the default branch with a matrix that checks out multiple branches.

## GitHub Actions Minutes and Billing

ADO provides a pool of parallel jobs (1 free Microsoft-hosted, 1 free self-hosted). GitHub provides a monthly minutes allotment that varies by plan — and multipliers apply for Windows (2x) and macOS (10x) runners.

**Impact:** Pipelines that were "free" on ADO self-hosted agents may incur significant cost on GitHub-hosted runners. Audit runner usage with `gh actions-importer forecast` and plan self-hosted runner infrastructure before migration.

## PR Status Checks Name Mapping

ADO build validation policies reference pipelines by definition ID. GitHub required status checks reference checks by *name string*. If you rename a job or workflow, the required check stops matching and PRs are blocked.

**Impact:** Lock down job names early and document them. Use explicit `name:` properties on jobs rather than relying on auto-generated names. Update branch protection rules whenever job names change.
