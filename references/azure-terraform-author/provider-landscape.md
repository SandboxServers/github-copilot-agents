## Provider Landscape

### azurerm Provider
- Primary provider for Azure resources — `hashicorp/azurerm`
- Covers most GA Azure services with HCL-native arguments
- Version constraint: always pin with `~>` (e.g., `~> 4.0`) — never use `>=` alone
- Features block required in provider configuration
- Known quirks:
  - Some resources have `create_or_update` behavior — lifecycle changes may trigger replacement unexpectedly
  - `ignore_changes` frequently needed for properties Azure mutates outside Terraform (e.g., tags set by policy, system-managed fields)
  - Deprecated properties linger for major version cycles — check changelog before using `enable_*` vs `*_enabled` naming
  - Some resources split create/update functions to fix lifecycle issues (VNet, VPN, backup resources)

### azapi Provider
- Thin layer over Azure ARM REST APIs — `Azure/azapi`
- Use when: feature not yet in azurerm, preview services, day-zero support for new resources
- Key resources: `azapi_resource`, `azapi_update_resource`, `azapi_resource_action`, `azapi_data_plane_resource`
- Key data sources: `azapi_resource`, `azapi_resource_list`, `azapi_resource_id`, `azapi_client_config`
- Supports ALL Azure control plane services and ALL API versions
- Built-in preflight validation
- Migration path: `aztfmigrate` tool converts azapi → azurerm when provider catches up
- Can be used alongside azurerm in the same configuration

### When to Use Which Provider
```
Is the resource/feature in azurerm GA?
├─ Yes → Use azurerm
├─ No — Is it in azurerm preview/beta?
│  ├─ Yes, and stable enough → Use azurerm
│  └─ No or too buggy → Use azapi
├─ Brand new service, not in azurerm at all? → Use azapi
└─ Need specific API version control? → Use azapi
```
