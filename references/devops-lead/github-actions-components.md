## GitHub Actions — Reusable Components at Scale

### Component Types

1. **Reusable workflows** (`workflow_call`): Central versioned workflow definitions invoked from other workflows. Define inputs, outputs, and secrets. The caller workflow delegates entire jobs.
2. **Composite actions**: Encapsulated step logic bundled as a single action. Combine multiple steps into one reusable unit with inputs and outputs.
3. **Starter workflows**: Template scaffolds that copy into new repos. Independent after creation — not centrally versioned like reusable workflows.

### Key Distinctions

| Feature | Reusable Workflow | Composite Action | Starter Workflow |
|---------|------------------|-----------------|-----------------|
| Invocation | `uses: org/repo/.github/workflows/x.yml@ref` | `uses: org/repo/action@ref` | Copied into repo on creation |
| Scope | Full jobs | Steps within a job | Full workflow scaffold |
| Central control | Yes — changes propagate | Yes — changes propagate | No — independent after copy |
| Versioning | Git ref (tag/branch/SHA) | Git ref (tag/branch/SHA) | N/A after creation |

### Enterprise GitHub Actions Management

- **Organization-level policies**: Control which actions are allowed (all, local only, selected)
- **Runner groups**: Manage self-hosted and GitHub-hosted runners across org/enterprise
- **IP allow lists and networking**: Restrict runner network access
- **Encrypted secrets**: Organization and repository level, environment-scoped
- **OIDC for Azure**: Workload identity federation eliminates stored credentials for Azure deployments
