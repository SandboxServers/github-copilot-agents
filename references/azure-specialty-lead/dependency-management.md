## Dependency Management

Azure resources have deployment dependencies. Deploy in the wrong order and you get failures, workarounds, or resources that can't connect to each other. The Specialty Lead owns the dependency graph.

### Standard Deployment Order

Deploy in this sequence. Each phase depends on the phases above it:

```
Phase 1: Foundation
  └── Resource groups
  └── Networking (VNets, subnets, NSGs, private DNS zones, peering)
  └── Identity (managed identities, Key Vault, RBAC baseline)

Phase 2: Data Layer
  └── Storage accounts (with private endpoints)
  └── Databases (SQL, Cosmos DB, PostgreSQL — with private endpoints)
  └── Data platform resources (Data Factory, Synapse, Fabric)

Phase 3: Compute
  └── App Service plans / AKS clusters / Container App environments
  └── App deployments (with VNet integration, managed identity bindings)
  └── Function Apps (with VNet integration, Key Vault references)

Phase 4: Integration
  └── Service Bus / Event Hubs / Event Grid
  └── API Management (with VNet integration)
  └── Logic Apps / Durable Functions

Phase 5: Observability (parallel with phases 1-4)
  └── Log Analytics workspace (deploy in phase 1)
  └── Application Insights (deploy with compute)
  └── Diagnostic settings (deploy with each resource)
  └── Alerts and dashboards (deploy after baseline established)
```

### Circular Dependency Avoidance

Circular dependencies indicate a design problem, not a deployment problem:

- **App needs database connection string → database needs app's managed identity for RBAC.** Solution: Deploy database first with a temporary admin. Deploy app with managed identity. Add RBAC assignment. Remove temporary admin.
- **API Management needs backend URL → backend needs APIM IP for firewall.** Solution: Deploy APIM first (URL is deterministic). Deploy backend with APIM's known IP in firewall rules.
- **Key Vault needs network rules → network rules need Key Vault for secrets.** Solution: Deploy Key Vault with public access initially. Add secrets. Configure private endpoint and disable public access.

The pattern: deploy with a temporary relaxed configuration, complete the dependency chain, then tighten.

### Terraform / Bicep Dependency Graph

Infrastructure-as-code tools manage dependencies declaratively:

- **Terraform:** Use `depends_on` only when implicit dependencies (via resource references) aren't sufficient. Over-using `depends_on` serializes deployments unnecessarily.
- **Bicep:** Use `dependsOn` for explicit ordering. Module outputs feed into downstream module inputs for implicit dependencies.
- **Both:** Structure modules by deployment phase. Network module → Data module → Compute module → Integration module. Each module's outputs are the next module's inputs.

### Post-Deployment Validation

After deployment, validate the dependency chain actually works:

1. **Network connectivity:** Can compute resources reach databases and storage via private endpoints? DNS resolution returning private IPs?
2. **Identity:** Can managed identities access Key Vault secrets? Can apps authenticate to databases?
3. **Data flow:** Can integration resources (Service Bus, Event Hubs) receive and deliver messages?
4. **Monitoring:** Are diagnostic settings flowing data to Log Analytics? Are Application Insights receiving telemetry?

Automate these checks. A deployment that completes without errors but can't pass connectivity validation is not a successful deployment.

### The Specialty Lead's Role

The Specialty Lead maintains the dependency graph across all specialists. When a specialist adds a new resource, the Specialty Lead identifies where it fits in the deployment order, what it depends on, and what depends on it. Missed dependencies surface as deployment failures or — worse — resources that deploy successfully but can't communicate.
