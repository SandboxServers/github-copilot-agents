# Authentication & Network Security for AI Services

## API Key Authentication

- **How it works**: Each Azure OpenAI resource has two API keys (primary and secondary) for key rotation
- **Usage**: Pass key in `api-key` HTTP header or SDK configuration
- **Scope**: Per-resource — one key grants access to all deployments on that resource
- **When to use**: Development, testing, quick prototyping only
- **Why not production**: Keys are static secrets that can leak, can't be scoped per-user, no audit trail per-caller, rotation requires application redeployment
- **If you must**: Store in Azure Key Vault, never in code or config files, rotate regularly

## Microsoft Entra ID (Managed Identity) — Preferred for Production

Managed Identity eliminates secrets entirely. The Azure platform handles token acquisition automatically.

### System-Assigned Managed Identity
- **Lifecycle**: Created with and tied to the Azure resource (e.g., App Service, Function App, VM)
- **Deleted when**: The parent resource is deleted
- **Best for**: Single-resource deployments, simple architectures
- **Limitation**: One identity per resource, can't be shared

### User-Assigned Managed Identity
- **Lifecycle**: Independent Azure resource — created, managed, and deleted separately
- **Shareable**: Can be assigned to multiple resources
- **Best for**: Multi-resource architectures, shared identity across services, lifecycle independence
- **Recommendation**: Prefer user-assigned for production — survives resource recreation during deployments

### Required RBAC Roles

| Role | Scope | Grants |
|------|-------|--------|
| Cognitive Services OpenAI User | Resource/RG/Sub | Invoke models (chat, completions, embeddings) |
| Cognitive Services OpenAI Contributor | Resource/RG/Sub | Deploy models, manage deployments + invoke |
| Cognitive Services User | Resource/RG/Sub | Invoke any Cognitive Services API |
| Cognitive Services Contributor | Resource/RG/Sub | Full management of Cognitive Services resources |
| Search Index Data Reader | AI Search resource | Read search indexes (for RAG) |
| Search Index Data Contributor | AI Search resource | Read/write search indexes |

**Principle of least privilege**: Use `Cognitive Services OpenAI User` for application code that only needs to call models. Use Contributor roles only for deployment automation.

## Entra ID Token-Based Auth (Application Code)

For applications authenticating with Entra ID using the Azure Identity SDK.

### DefaultAzureCredential Pattern
The recommended approach — chains through multiple credential types automatically:

```python
from azure.identity import DefaultAzureCredential
from openai import AzureOpenAI

credential = DefaultAzureCredential()
token = credential.get_token("https://cognitiveservices.azure.com/.default")

client = AzureOpenAI(
    azure_endpoint=endpoint,
    azure_ad_token=token.token,
    api_version="2024-10-21"
)
```

### Credential Chain Order
`DefaultAzureCredential` tries these in order:
1. Environment variables (service principal)
2. Workload Identity (AKS)
3. Managed Identity (system-assigned, then user-assigned)
4. Azure CLI credential (developer machines)
5. Azure PowerShell credential
6. Interactive browser (fallback)

**In production**: Managed Identity is picked up automatically — no configuration needed.
**In development**: Azure CLI login (`az login`) is used automatically.

### Service Principal (App Registration)
- For scenarios where Managed Identity isn't available (cross-tenant, external services)
- Create App Registration in Entra ID, assign RBAC roles
- Authenticate with client ID + client secret or certificate
- **Prefer certificates** over client secrets for production

## Virtual Network & Private Endpoints

### Private Endpoints for Azure OpenAI
- Deploy a private endpoint for your Azure OpenAI resource into your VNet
- Traffic flows over Microsoft backbone network, never the public internet
- **Disable public network access** for production resources after configuring private endpoints

### Architecture Pattern
```
[App in VNet] → [Private Endpoint] → [Azure OpenAI (public access disabled)]
                                   → [Azure AI Search (public access disabled)]
                                   → [Azure Key Vault (public access disabled)]
```

### Private DNS Zones
- Required for private endpoint name resolution
- Create private DNS zone: `privatelink.openai.azure.com`
- Link to VNet for automatic DNS resolution
- Without proper DNS, private endpoints won't resolve correctly

### NSG Rules
- Network Security Groups control traffic flow to/from subnets
- Allow outbound HTTPS (443) from app subnet to private endpoint subnet
- Deny all other outbound traffic for defense in depth
- Log NSG flow logs for audit trail

## Customer-Managed Keys (CMK)

- Azure OpenAI encrypts data at rest with Microsoft-managed keys by default
- Enable customer-managed keys (CMK) for regulatory requirements
- Keys stored in Azure Key Vault (or Managed HSM for FIPS 140-2 Level 3)
- Applies to: fine-tuning data, uploaded files, stored completions
- **Note**: Does not affect model inference — only data at rest

## Network Security Checklist for Production

1. Deploy Azure OpenAI with private endpoint in your VNet
2. Disable public network access on the Azure OpenAI resource
3. Configure private DNS zones for name resolution
4. Use Managed Identity (user-assigned) for authentication — no API keys
5. Assign minimum required RBAC roles (`Cognitive Services OpenAI User`)
6. Enable diagnostic logging to Log Analytics workspace
7. Configure NSG rules to restrict network access
8. Enable customer-managed keys if required by compliance
9. Review and restrict the resource's network ACLs
10. Monitor with Azure Monitor alerts for authentication failures and throttling
