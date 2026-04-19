## Implementation Options

| Accelerator | Tooling | Best For | Notes |
|-------------|---------|----------|-------|
| ALZ Portal Accelerator | Azure Portal wizard | Initial greenfield deployment, demo | Not repeatable, not IaC |
| ALZ Bicep | Bicep modules (`Azure/ALZ-Bicep`) | Organizations using Bicep/ARM | Modular, AVM-aligned |
| ALZ Terraform | Terraform module (`Azure/terraform-azurerm-caf-enterprise-scale`) | Organizations using Terraform | Full ALZ in single module |
| Sovereign Landing Zone | Bicep | Government/sovereign cloud | Adds compliance controls |
| Custom | Org-specific IaC wrapping AVM modules | Advanced orgs with specific needs | Most flexible, most effort |
