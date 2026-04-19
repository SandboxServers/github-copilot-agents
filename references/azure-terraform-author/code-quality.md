## Code Quality Checklist

### Every Module Should Have
- [ ] `terraform.tf` with pinned provider versions (`~>` constraint)
- [ ] Variables with `description`, `type`, `default` (where sensible), and `validation` (where constrained)
- [ ] Outputs for resource `id`, `name`, and any connection/endpoint info
- [ ] `lifecycle` blocks on stateful resources (`prevent_destroy`)
- [ ] `ignore_changes` for Azure-managed properties
- [ ] Examples that deploy without manual input
- [ ] At least one `.tftest.hcl` test file
- [ ] No hardcoded values — everything parameterized or computed in `locals`
- [ ] No provider blocks (only `required_providers` with `configuration_aliases` if needed)
- [ ] Consistent naming in locals block, not scattered across resources

### Every Root Module Should Have
- [ ] Remote backend configuration
- [ ] Variable definitions file per environment (`.tfvars`)
- [ ] Lock file committed (`.terraform.lock.hcl`)
- [ ] `.gitignore` excluding `.terraform/`, `*.tfstate`, `*.tfstate.backup`, `*.tfplan`
- [ ] CI pipeline running: fmt → validate → tflint → checkov → plan → apply (with approval gate)

## Anti-Patterns to Reject

1. **Monolithic state** — one state file for everything → blast radius of one bad apply = entire infrastructure
2. **Hardcoded subscription/tenant IDs** — use data sources or variables
3. **count with complex conditionals** — use `for_each` with maps for anything non-trivial
4. **Secrets in terraform.tfvars** — use Key Vault data source or environment variables
5. **Nested modules more than 2 levels** — if your module calls a module that calls a module, it's too deep
6. **Over-abstracted modules** — a module that wraps a single resource with 40 variables is worse than the resource
7. **No lock file committed** — dependency drift between team members
8. **apply -auto-approve in production** — always plan → review → apply with explicit approval
9. **Ignoring plan output** — if plan says "1 to destroy, 1 to create" on a database, STOP
10. **Using `terraform workspace` for isolation in production** — workspaces share backend, not truly isolated
