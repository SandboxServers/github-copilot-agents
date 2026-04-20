# ADO → GitHub Migration Best Practices

## Migration Waves: Lowest Risk First

Group migrations by team or project boundary. Start with a low-complexity, low-dependency project to build confidence and refine the process. Each wave should be 1-3 teams with a 2-week execution window.

**Wave structure:**
1. Audit and inventory the wave's repos, pipelines, and dependencies
2. Migrate repos and configure branch protection
3. Convert pipelines and validate in non-production branches
4. Parallel operation for 2-4 weeks
5. Cutover: disable ADO triggers, enable GitHub triggers
6. Retrospective before next wave

## Parallel Operation Period

Run both ADO and GitHub pipelines simultaneously during transition. ADO remains the system of record until GitHub is validated end-to-end through production deployment.

**Key rules:**
- ADO pipeline continues to deploy to production
- GitHub pipeline deploys to a staging/validation environment
- Compare build artifacts, test results, and deployment outcomes
- Only cut over when GitHub produces identical results for 2+ successful cycles

## Audit Before You Migrate

Inventory everything. Identify what's active, what's stale, and what's dead.

- Run `gh actions-importer audit azure-devops` for pipeline assessment
- Check last-commit dates on all repos — archive anything untouched for 12+ months
- Check last-run dates on pipelines — decommission anything not run in 6+ months
- Consolidate duplicate variable groups and remove unused service connections
- Document team ownership for every repo and pipeline

## Service Connections → OIDC Federation

Replace long-lived service principal secrets with OIDC federated credentials using Microsoft Entra Workload Identity Federation. This eliminates secret rotation and reduces credential sprawl.

**Per service connection:**
1. Create a federated credential on the existing app registration
2. Set subject claim: `repo:<org>/<repo>:environment:<env-name>`
3. Use `azure/login@v2` with `client-id`, `tenant-id`, `subscription-id`
4. Remove the old client secret from both ADO and Entra ID

See `service-connection-to-oidc.md` for detailed patterns and subject claim options.

## Redesign Template Architecture

Do not translate ADO templates 1:1 to GHA. ADO's template model (with conditional insertion, variable expansion, and nested templates) is fundamentally different from GHA's reusable workflows and composite actions.

**Migration mapping:**
- ADO stage templates → Reusable workflows (`workflow_call`)
- ADO job/step templates → Composite actions
- ADO variable templates → Organization/repo variables or env files
- ADO `extends` templates → Called workflows with required inputs

Centralize reusable workflows in a dedicated `.github` repository with proper versioning via tags.

## Communication Plan

Migration affects every developer touching the migrated projects. Proactive communication prevents confusion and resistance.

**Minimum communication:**
- Announce migration timeline 4+ weeks before each wave
- Provide training materials: GHA syntax, OIDC setup, PR workflow, branch protection
- Share a migration FAQ with common questions and links to documentation
- Designate a migration champion per team for first-response support
- Send daily status updates during active migration windows

## Validate Migrated Pipelines

Migrated pipelines must produce the same outputs as their ADO equivalents. Bit-for-bit artifact comparison isn't always possible, but functional equivalence is mandatory.

**Validation checklist:**
- Build artifacts are the same version, contain the same files
- Unit tests pass with the same results
- Integration/deployment tests reach the same endpoints
- Deployment to staging succeeds with the correct configuration
- Notifications and status checks report correctly
- Secrets and variables resolve to correct values

## Migration Runbook

Document every step of the migration process in a runbook. This is the single source of truth during execution.

**Runbook sections:**
- Pre-migration checklist (audit complete, training done, stakeholders informed)
- Repository migration steps (clone, clean history, push, configure protections)
- Pipeline conversion steps (importer run, manual fixes, validation)
- Secret/variable migration steps (scope mapping, Key Vault integration)
- Environment setup steps (protection rules, required reviewers)
- Cutover steps (disable ADO triggers, enable GitHub, DNS/webhook changes)
- Rollback steps (re-enable ADO, disable GitHub, communication plan)

## Post-Migration Decommission

After successful cutover and validation period, decommission ADO resources on a documented schedule.

- Set ADO repos to read-only (do not delete immediately — preserve history access)
- Disable ADO pipeline triggers (keep definitions for reference)
- Remove ADO service connection secrets (after OIDC is confirmed working)
- Archive ADO project after 90 days of successful GitHub operation
- Retain ADO work item access for historical reference (read-only)
