## ADO to GitHub Migration — Platform Transitions

### Migration Scope

| ADO Component | GitHub Equivalent | Migration Complexity |
|---------------|-------------------|---------------------|
| Azure Repos (Git) | GitHub Repositories | Low — direct Git migration |
| Azure Repos (TFVC) | GitHub Repositories | Medium — requires conversion to Git |
| Azure Pipelines (YAML) | GitHub Actions | Medium — syntax translation, different expression language |
| Azure Pipelines (Classic) | GitHub Actions | High — no direct mapping, full rewrite |
| Azure Boards | GitHub Issues/Projects | Medium — different models, work item mapping needed |
| Azure Artifacts | GitHub Packages | Medium — different feed models |
| Azure Test Plans | Third-party or custom | High — no direct equivalent |

### Migration Strategy

1. **Assess**: Inventory all ADO assets, identify dependencies, map to GitHub equivalents
2. **Plan**: Choose migration order (repos first, then pipelines, then boards), set cutover timeline
3. **Pilot**: Migrate one representative project end-to-end, document lessons
4. **Execute**: Migrate in waves, set source to read-only after cutover
5. **Validate**: Verify builds pass, deployments work, integrations connect

### Key Considerations

- **Tip migration**: Migrate latest code only — leave history in ADO (keep ADO read-only for reference)
- **Branch strategy**: Migration is an opportunity to simplify branching — move to trunk-based development
- **Service connections → OIDC**: Replace ADO service connections with GitHub OIDC workload identity federation
- **Binary files**: Remove from repo, move to Git LFS or package feeds
- **Breadcrumbs**: Leave pointers in new Git repo back to old ADO system for history lookup
