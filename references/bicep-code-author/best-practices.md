## Naming Conventions and Best Practices

### File Organization
```
infra/
├── main.bicep              # Entry point
├── main.bicepparam         # Parameter file (per environment)
├── bicepconfig.json        # Linter and registry configuration
├── modules/
│   ├── networking.bicep    # Custom module
│   └── compute.bicep       # Custom module
└── tests/
    └── main.test.bicep     # Deployment tests
```

### Secure Parameter Handling
1. Mark secrets with `@secure()` — never provide defaults for secure params
2. Reference Key Vault in parameter files — `az.getSecret()` in bicepparam or `reference` in JSON
3. Never output secrets — linter rule `outputs-should-not-contain-secrets` catches this
4. Use `@secure()` on output only when absolutely necessary and document why

### Code Quality
- Use `@description()` on all params and outputs
- Prefer string interpolation over `concat()`
- Use resource symbolic names for references instead of `reference()` / `resourceId()`
- Use `existing` keyword instead of hardcoded resource IDs
- Use `parent` property for child resources instead of nested `name` with `/`
- Remove unused params, vars, and imports — linter enforces this
- Pin API versions — use latest stable, not preview unless needed
