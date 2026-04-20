# ADO → GitHub Migration Anti-Patterns

## Big Bang Migration

Migrating every repo, pipeline, and artifact feed in a single cutover weekend. Inevitably something breaks, nobody knows which system is authoritative, and rollback affects everything.

**Why it fails:** Cross-team dependencies surface mid-migration. Shared template repos break pipelines in other projects. Artifact feed switches break consumers who haven't updated their sources.

**Fix:** Migrate in waves grouped by team or project. Start with lowest-risk, lowest-dependency projects. Each wave has its own validation window. Limit each wave to 2-3 teams maximum.

## 1:1 Pipeline Translation

Directly converting ADO YAML syntax to GitHub Actions YAML line-by-line. ADO and GHA have fundamentally different models — task groups ≠ composite actions, stages ≠ jobs, and trigger models differ significantly.

**Common symptoms:** Workflows that shell out to replicate ADO task behavior instead of using native actions. Stage-based YAML that maps each ADO stage to a separate workflow file for no reason. Trigger configurations that try to replicate ADO's pipeline resource triggers verbatim.

**Fix:** Treat migration as a redesign opportunity. Map the *intent* of each pipeline, then implement idiomatically in GHA using reusable workflows, composite actions, and native features like OIDC.

## Migrating Without an Audit

Moving 200 repos and 150 pipelines without checking which are actually in use. You end up maintaining dead pipelines and abandoned repos in GitHub.

**Fix:** Run `gh actions-importer audit` against ADO. Cross-reference with last-commit dates, last-run dates, and team ownership. Archive or delete anything unused *before* migrating.

## Ignoring Parity Gaps

Assuming GitHub has a direct equivalent for every ADO feature. It doesn't — deployment strategies, gates, test plans, and release pipeline approval chains all have gaps.

**Fix:** Conduct a parity gap assessment using `parity-gap-workarounds.md`. Document every gap and its workaround *before* committing to migration timelines. Surface gaps to stakeholders early.

## Migrating Service Connections Without Review

Copying every service connection's credentials into GitHub Secrets without reviewing permission scope, expiry, or whether the connection is even needed.

**Fix:** Audit service connections for least-privilege. Migrate to OIDC/workload identity federation. Remove unused connections. Review subscription-level vs. resource-group-level scope.

## No Parallel Operation Period

Cutting over from ADO to GitHub in one shot with no overlap. If anything breaks, there's no fallback and deployments stall.

**Fix:** Run both systems in parallel for 2-4 weeks per wave. ADO remains the system of record until the GitHub pipeline is validated end-to-end including production deployment.

## Skipping Variable Group Consolidation

Migrating 30 variable groups when 10 of them are duplicates or near-duplicates across different pipelines. GitHub has no variable groups — every secret/variable is managed at org, repo, or environment level.

**Fix:** Consolidate variable groups *before* migration. Deduplicate, remove stale values, and plan the target scope (org-level secret, repo-level variable, environment secret) for each value.

## Not Training Teams Before Migration

Switching teams to GitHub without training on GHA syntax, PR workflows, branch protection rules, or the GitHub security model. Teams flounder and revert to anti-patterns.

**Fix:** Run hands-on workshops covering GHA workflow syntax, OIDC setup, environment protections, and PR review workflows *before* the team's migration wave.

## Migrating Test Plans Without a Strategy

ADO Test Plans have no native equivalent in GitHub. Attempting to migrate test cases, test suites, and test runs without deciding on a replacement leads to data loss or orphaned artifacts.

**Fix:** Choose a destination: GitHub Issues with labels, a third-party test management tool (Zephyr, TestRail), or markdown-based test documentation. Map test plan structure to the chosen target *before* migrating.

## Automated Migration of All Work Items

Attempting to script-migrate every work item, including closed items from years ago, into GitHub Issues. The result is thousands of stale issues polluting the backlog.

**Fix:** Migrate only active work items. Archive historical items in ADO (set to read-only) and link to them from GitHub. For active items, use `gh` CLI or the GitHub API to create issues with proper labels and milestones.

## No Rollback Plan

Starting a migration wave without a documented rollback procedure. If pipeline X fails in GitHub, nobody knows whether to fix forward or revert to ADO.

**Fix:** Document a per-wave rollback plan: what triggers rollback, who decides, how to revert DNS/branch-protection/trigger changes, and how to re-enable ADO pipelines.

## Using GitHub Importer for Complex Repos Without Testing

Running `gh actions-importer` on complex pipelines with heavy template usage, conditional insertion, and custom tasks — then trusting the output without review.

**Fix:** Always run `--dry-run` first. Review the conversion output for TODO comments and unsupported features. Test the generated workflow in a non-production branch before merging. Expect 60-80% automated conversion on complex pipelines — the rest is manual.

## Copying ADO Project Structure into GitHub

Replicating ADO's project/team/area-path hierarchy as GitHub organizations, teams, and repos. GitHub's organizational model is flatter — forcing ADO's structure creates unnecessary complexity.

**Fix:** Map ADO projects to GitHub repositories or repository groups. Use GitHub teams for access control. Use labels or project boards instead of area paths. Accept that the organizational model will look different.

## Neglecting GitHub Security Features

Migrating pipelines without enabling Dependabot, secret scanning, code scanning, or branch rulesets. ADO has different security defaults — the GitHub equivalents must be explicitly configured.

**Fix:** Enable security features as part of each migration wave. Configure Dependabot for dependency updates, enable secret scanning with push protection, set up CodeQL for code scanning, and create branch rulesets before the first PR lands.
