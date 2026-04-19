## Subscription Vending

### What It Is
Self-service (or near-self-service) process for application teams to request and receive Azure subscriptions pre-configured with:
- Correct management group placement
- Network connectivity (VNet peered to hub, DNS configured)
- RBAC assignments for the requesting team
- Azure Policy inherited from management group
- Diagnostic settings configured
- Budget alerts set

### Vending Product Lines

| Product Line | Management Group | Network | Use Case |
|-------------|-----------------|---------|----------|
| **Corp workload** | Landing Zones → Corp | VNet peered to hub, private connectivity | Internal LOB apps, databases, APIs |
| **Online workload** | Landing Zones → Online | VNet peered to hub, public endpoints allowed | Customer-facing web apps, APIs |
| **Sandbox** | Sandbox | Isolated VNet (no hub peering) | Experimentation, learning, PoCs |
| **Shared platform** | Landing Zones → Corp | VNet peered to hub, special RBAC | AKS clusters, shared data platforms |
| **SAP** | Landing Zones → Corp (or dedicated MG) | VNet peered to hub, specific network rules | SAP HANA, S/4HANA deployments |

### Implementation Options

| Tool | Repository | Notes |
|------|-----------|-------|
| Bicep subscription vending | `Azure/bicep-registry-modules` (AVM pattern `lz/sub-vending`) | Official AVM module |
| Terraform subscription vending | `Azure/lz-vending` on Terraform Registry | Official AVM module |
| ALZ Portal Accelerator | Azure Portal deployment experience | Good for initial setup, not ongoing vending |
| Custom automation | Org-specific pipelines | Wraps Bicep/Terraform modules with org-specific logic |
