# GitHub Actions Importer — Advanced Usage

## Custom Transformers

The GitHub Actions Importer supports custom transformers written in Ruby to handle ADO tasks
that have no built-in mapping or to override default conversion behavior.

### File Structure

Custom transformers live in a directory you specify with `--custom-transformers`:

```
custom-transformers/
├── transformers/
│   ├── my_custom_task.rb          # Maps a specific ADO task
│   ├── override_dotnet_build.rb   # Overrides a built-in mapping
│   └── shared_helpers.rb          # Shared utility methods
└── env.yml                         # Optional environment config
```

### Writing a Custom Transformer

Each transformer is a Ruby file that calls `transform` with the ADO task identifier:

```ruby
# transformers/my_custom_task.rb
transform "MyOrg.CustomDeploy@1" do |task|
  # task[:inputs] contains ADO task inputs as a hash
  target_env = task[:inputs]["environment"]
  package_path = task[:inputs]["packagePath"]

  # Return a GitHub Actions step hash
  {
    name: "Deploy to #{target_env}",
    uses: "azure/webapps-deploy@v3",
    with: {
      "app-name" => target_env,
      "package" => package_path
    }
  }
end
```

### Overriding Built-in Transformers

To change how the importer handles a known task (e.g., DotNetCoreCLI@2):

```ruby
transform "DotNetCoreCLI@2" do |task|
  command = task[:inputs]["command"]
  projects = task[:inputs]["projects"]

  # Custom mapping: always use setup-dotnet + run instead of default
  [
    { name: "Setup .NET", uses: "actions/setup-dotnet@v4", with: { "dotnet-version" => "8.0.x" } },
    { name: "Run dotnet #{command}", run: "dotnet #{command} #{projects}" }
  ]
end
```

### Passing Custom Transformers to Importer

```bash
gh actions-importer dry-run azure-devops \
  --custom-transformers ./custom-transformers/transformers/ \
  --output-dir ./output \
  --azure-devops-organization my-org \
  --azure-devops-project my-project \
  --pipeline-id 42
```

## Importer Configuration

The importer reads `.github/actions-importer/config.yml` for default settings:

```yaml
# .github/actions-importer/config.yml
source:
  type: azure_devops
  organization: my-org
  project: my-project
  access_token: ${ADO_PAT}

target:
  type: github
  organization: my-github-org
  access_token: ${GITHUB_TOKEN}

options:
  # Prefix for all converted workflow files
  workflow_file_prefix: "migrated-"
  # Default runner label for converted workflows
  runner_label: ubuntu-latest
```

## Understanding Importer Output Comments

The importer annotates generated workflows with structured comments:

| Comment Prefix | Meaning | Action Required |
|---|---|---|
| `# Unsupported:` | ADO feature has no GHA equivalent | Manual redesign needed |
| `# Manual:` | Conversion possible but needs human decision | Review and implement |
| `# Task:` | ADO task reference for context | Informational only |
| `# TODO:` | Known gap the importer flagged | Must address before use |

Example output with annotations:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # Task: DotNetCoreCLI@2 (restore)
      - name: Restore dependencies
        run: dotnet restore
      # Unsupported: Pipeline decorators are not supported in GitHub Actions
      # Manual: Convert ADO predefined variable Build.Repository.LocalPath
      - name: Build
        run: dotnet build --configuration Release
```

## Audit Report Deep-Dive

Run `gh actions-importer audit azure-devops` to generate `audit_summary.md`:

### Key Sections in audit_summary.md

- **Pipelines Summary**: Total pipelines scanned, conversion success rate, partial conversions
- **Task Usage**: Frequency table of every ADO task across all pipelines — identifies common tasks to prioritize custom transformers for
- **Unsupported Features**: Aggregated list of features that need manual migration
- **Manual Steps Required**: Items needing human review (service connections, secrets, etc.)
- **Runner Mapping**: Summary of agent pool → runner label conversions needed
- **Environment References**: All ADO environments referenced across pipelines

### Interpreting Conversion Metrics

- **Fully converted**: Workflow should work with minimal changes
- **Partially converted**: Core structure migrated, some steps need manual work
- **Not converted**: Pipeline uses features the importer cannot handle

## Forecast Report Analysis

The forecast command estimates GitHub Actions runner usage:

```bash
gh actions-importer forecast azure-devops \
  --azure-devops-organization my-org \
  --azure-devops-project my-project \
  --start-date 2025-01-01 --end-date 2025-03-31
```

The forecast report provides:
- **Total runner hours** by pipeline (maps to GitHub Actions billing minutes)
- **Peak concurrency** — maximum parallel jobs (helps size self-hosted runner fleet)
- **Job duration percentiles** (p50, p90, p99) for capacity planning
- **Queue time analysis** — helps identify if more runners are needed
- **OS breakdown** — Linux vs Windows minutes for cost estimation (Windows is 2x cost)

## Batch Migration

Migrate all pipelines in a project at once:

```bash
gh actions-importer migrate azure-devops \
  --azure-devops-organization my-org \
  --azure-devops-project my-project \
  --target-url https://github.com/my-github-org/my-repo \
  --output-dir ./migration-output
```

Output directory structure:
```
migration-output/
├── pipeline-1/
│   ├── .github/workflows/pipeline-1.yml
│   └── pipeline-1.log
├── pipeline-2/
│   ├── .github/workflows/pipeline-2.yml
│   └── pipeline-2.log
└── audit_summary.md
```

## Known Importer Limitations

These ADO features have NO automated conversion path:

| ADO Feature | Why Unsupported | Migration Approach |
|---|---|---|
| `${{ each }}` nested blocks | GHA has no `each` equivalent | Use matrix strategy or duplicate steps |
| `elseif` conditionals | GHA only supports `if` (no elseif chain) | Restructure as separate `if` conditions |
| Pipeline decorators | Org-level injection concept doesn't exist | Use reusable workflows or required workflows |
| TFVC triggers | GHA only supports Git | Migrate source to Git first |
| Pre/post deployment gates | Classic release concept | Use environment protection rules |
| Conditional compilation (`$[variables.X]`) | Compile-time vs runtime difference | Use `if:` conditions with `vars.` context |
| `settableVariables` restrictions | No GHA equivalent | Remove or use environment-level controls |
| Demands / capabilities | Agent capability matching | Use runner labels and `runs-on` |

## Post-Importer Cleanup Checklist

After automated conversion, always verify these items:

1. **Service connections** → Replace with OIDC credentials or GitHub secrets
2. **Variable groups** → Migrate to repository/org/environment variables and secrets
3. **Agent pools** → Update `runs-on` labels for your runner fleet
4. **Checkout paths** → ADO uses `$(Build.SourcesDirectory)`, GHA uses `${{ github.workspace }}`
5. **Artifact paths** → Replace `PublishPipelineArtifact` with `actions/upload-artifact`
6. **Condition syntax** → ADO `condition:` uses different function syntax than GHA `if:`
7. **Predefined variables** → Map `$(Build.BuildNumber)` etc. to `${{ github.run_number }}`
8. **Template references** → Convert `template@repo` to reusable workflow or composite action
9. **Environment references** → Create matching GitHub environments with protection rules
10. **Secret references** → All `$(secret-var)` must become `${{ secrets.SECRET_VAR }}`

## Debug Mode

Run the importer with verbose output for troubleshooting:

```bash
gh actions-importer dry-run azure-devops \
  --azure-devops-organization my-org \
  --azure-devops-project my-project \
  --pipeline-id 42 \
  --output-dir ./debug-output \
  2>&1 | tee importer-debug.log
```

Set `ACTIONS_IMPORTER_LOG_LEVEL=DEBUG` for maximum verbosity. Review the log for:
- Task mapping decisions (which transformer was selected)
- Parameter resolution failures
- Template expansion issues
- API call failures to ADO REST endpoints

## Version Management

Always test with dry-run before upgrading the importer:

```bash
# Check current version
gh actions-importer version

# Update to latest
gh actions-importer update

# Always dry-run after update to verify no regressions
gh actions-importer dry-run azure-devops --pipeline-id 42 --output-dir ./test-output
```

Compare dry-run output before and after upgrade using diff tools to catch behavioral changes.
