## Common azurerm Gotchas

### Resource Replacement Triggers (Things That Force Destroy/Recreate)
- Changing `name` on most resources
- Changing `location` on any resource
- Changing `sku` on some resources (not all — check docs)
- Changing `os_type` on compute resources
- Changing `kind` on storage accounts, app services
- Adding/removing `identity` block on some resources

### Properties Azure Mutates (Need `ignore_changes`)
```hcl
# Tags set by Azure Policy
ignore_changes = [tags["CreatedDate"], tags["CreatedBy"]]

# AKS node count managed by autoscaler
ignore_changes = [default_node_pool[0].node_count]

# App Service configuration set by deployment
ignore_changes = [site_config[0].linux_fx_version]

# Cosmos DB throughput managed by autoscale
ignore_changes = [throughput]
```

### Naming Convention Anti-Patterns
- Don't embed resource type abbreviation inconsistently — pick a convention (CAF recommended: `rg-`, `st`, `kv-`, `aks-`) and enforce it
- Don't use underscores in resource names that map to DNS (storage accounts, key vaults)
- Storage account names: 3-24 chars, lowercase letters and numbers only, globally unique
- Key vault names: 3-24 chars, alphanumeric and hyphens, globally unique
