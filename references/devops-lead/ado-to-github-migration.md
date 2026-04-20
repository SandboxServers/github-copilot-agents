## ADO to GitHub Migration — Planning and Execution

Migrating from Azure DevOps to GitHub is a multi-phase effort that affects every engineering team. Success requires careful planning, clear feature mapping, and a phased approach that maintains productivity during transition.

### Migration Planning

Before migrating anything, establish the scope and timeline:
- **Inventory**: Catalog all ADO assets — repositories, pipelines, work items, test plans, artifacts, wikis, dashboards
- **Dependencies**: Map cross-repo and cross-project dependencies that constrain migration order
- **Team readiness**: Assess GitHub familiarity across teams — training investment may be required
- **Timeline**: Plan for 3–12 months depending on organization size. Rushing migrations creates technical debt
- **Success criteria**: Define what "done" means — all repos migrated, all pipelines running, ADO decommissioned

### Feature Mapping

| Azure DevOps | GitHub Equivalent | Notes |
|--------------|-------------------|-------|
| Azure Repos (Git) | GitHub Repositories | Direct migration with full history. Use GitHub Importer or `git push --mirror` |
| Azure Repos (TFVC) | GitHub Repositories | Requires conversion to Git first using `git-tfs` or similar tooling. Plan for history loss decisions |
| Azure Pipelines (YAML) | GitHub Actions | Conceptually similar but syntax differs. Rewrite required — no automated conversion |
| Azure Pipelines (Classic) | GitHub Actions | Must be converted to YAML first, then translated to Actions workflow syntax |
| Azure Boards | GitHub Issues + Projects | Different model — Issues are simpler than Work Items. Custom fields via Projects. Migration tooling exists but may lose fidelity |
| Azure Artifacts | GitHub Packages | NuGet, npm, Maven, Docker supported. Republish packages — no direct migration path |
| Azure Test Plans | No direct equivalent | GitHub has no built-in test management. Use third-party tools or adapt testing workflows |
| Azure Wikis | GitHub Wiki or repo docs | Simple conversion. Markdown-based wikis transfer cleanly |
| Azure Dashboards | GitHub Insights + third-party | Native dashboards are limited. Use GitHub API with external visualization tools |

### Repository Migration

For Git repositories, migration preserves full commit history:
1. Clone the ADO repo with `--mirror` to capture all branches and tags
2. Create the target GitHub repository
3. Push with `--mirror` to the GitHub remote
4. Update CI/CD references, submodule URLs, and documentation links
5. Set up branch protection rules equivalent to ADO branch policies
6. Redirect or archive the ADO repo after validation

For TFVC repositories, convert to Git before migrating. Decide whether to preserve full TFVC history (complex, slow) or take a tip snapshot (clean start, loses history). Most teams choose a tip snapshot with the TFVC repo archived for historical reference.

### Pipeline Migration

Pipeline migration is the most labor-intensive phase:
- There is no automated translation from Azure Pipelines YAML to GitHub Actions workflow syntax
- Map pipeline concepts: stages → jobs, task groups → reusable workflows, variable groups → repository/organization secrets and variables
- Rebuild shared templates as reusable workflows or composite actions
- Test pipelines thoroughly in parallel — run both ADO and GitHub pipelines during transition
- Migrate service connections to GitHub Environments with OIDC-based authentication

### Work Item Migration

GitHub Issues and Projects offer a simpler model than Azure Boards:
- Work Items become Issues — some field fidelity will be lost (custom fields, rich linking)
- Use GitHub Projects for board views, sprints, and custom fields
- Migration tools (gh CLI extensions, third-party importers) can automate bulk creation
- Preserve ADO Work Item IDs in Issue descriptions for cross-reference during transition
- Accept that the GitHub model is different — do not try to recreate Azure Boards in GitHub

### Phased Migration Approach

**Phase 1 — Foundation**: Set up GitHub organization, configure SSO/SCIM, establish repository naming conventions, create shared template repos, train early adopters

**Phase 2 — Pilot**: Migrate 2–3 non-critical repositories end-to-end. Validate the full workflow: code → build → test → deploy. Document lessons learned and refine the process

**Phase 3 — Migration Waves**: Migrate remaining repositories in planned waves grouped by team or domain. Each wave follows the validated process from the pilot

**Phase 4 — Decommission**: After all teams are on GitHub, archive ADO projects. Maintain read-only access for historical reference during a defined retention period

### Coexistence During Transition

During migration, teams will use both platforms simultaneously:
- Cross-platform links between ADO Work Items and GitHub Issues may be needed
- CI/CD may run in both platforms for the same repo during pipeline validation
- Establish clear communication about which platform is the source of truth for each repo
- Avoid the trap of maintaining both platforms indefinitely — set firm cutover dates per team