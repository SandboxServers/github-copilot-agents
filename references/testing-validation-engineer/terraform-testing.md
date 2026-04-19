### Terraform Testing

#### Static Analysis (Pre-Plan)
| Tool | Purpose | Speed |
|---|---|---|
| `terraform validate` | Syntax and internal consistency | Seconds |
| `terraform fmt` | Format compliance | Seconds |
| **tflint** (with azurerm ruleset) | Linting — deprecated arguments, invalid values, naming conventions | Seconds |
| **Checkov** | Security/compliance policy scanning — CIS, NIST, PCI-DSS checks | Seconds |
| **terrafmt** | Format validation for embedded HCL in documentation | Seconds |

#### Plan Validation (Pre-Apply)
| Technique | Purpose |
|---|---|
| `terraform plan` | Preview changes, identify drift, catch unexpected destroys |
| Plan assertions | Programmatic inspection of plan output — verify expected resource counts, no unexpected changes |
| terraform-compliance | BDD-style policy testing against plan output — "if you're creating an Azure resource, it must contain a tag" |
| Cost estimation | Infracost or similar to estimate cost impact of planned changes |

#### Integration Testing (Post-Apply)
| Tool | Purpose |
|---|---|
| `terraform test` | Native HCL-based test framework — create/validate/destroy in isolated environments |
| **Terratest** (Go) | Programmatic testing — deploy, validate with Azure SDK calls, destroy. Full lifecycle. |
| Azure Resource Graph | Query deployed resources to verify configuration matches specification |
| Policy compliance | Verify deployed resources satisfy Azure Policy assignments |

#### AVM Testing Requirements
Azure Verified Modules require: `terraform validate/fmt/test`, terrafmt, Checkov, tflint with azurerm ruleset. Use Go for additional custom tests.
