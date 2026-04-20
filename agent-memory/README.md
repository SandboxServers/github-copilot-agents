# Agent Knowledge Base — Reference Files

This folder contains the structured knowledge base that powers each agent in the organization. Every agent with domain-specific expertise has a corresponding subdirectory here containing chunked reference material that the agent loads on demand during conversations.

## Why This Structure

VS Code Copilot agents operate within a context window. Loading an entire knowledge base into every conversation would waste context on irrelevant material and crowd out the actual work. Instead, each agent's knowledge is split into focused topic files that can be loaded selectively based on what the user is actually asking about.

The architecture follows a **table-of-contents-first** pattern:

1. The agent's `.agent.md` instructions tell it to read `_toc.md` first
2. `_toc.md` is a lightweight index — topic names, filenames, and "when to load" guidance
3. The agent reads only the chunks relevant to the current question
4. This keeps context usage proportional to task complexity

This is a deliberate tradeoff: slightly more overhead per question (reading the TOC, then loading files) in exchange for dramatically better context efficiency across all questions. An agent answering a simple question loads one chunk. An agent working a complex design loads several. Neither loads the full set.

## Structure

```
agent-memory/
├── <agent-name>/
│   ├── _toc.md              # Knowledge index — always loaded first
│   ├── topic-a.md           # Focused chunk on a single topic
│   ├── topic-b.md           # Another focused chunk
│   └── ...
```

### `_toc.md` — The Knowledge Index

Every subdirectory has a `_toc.md` file that serves as the agent's map of its own knowledge. It contains:

- A **timestamp** indicating when the content was last updated
- A **table** mapping each topic to its filename and a plain-English description of when to load it
- An instruction to **not load all files at once**

The "When to Load" column is critical — it tells the agent under what circumstances each file is relevant, preventing unnecessary context consumption.

### Topic Files — The Chunks

Each `.md` file covers a single domain topic in depth. Files are sized to be useful in isolation — large enough to contain actionable guidance, small enough to fit comfortably in context alongside conversation and code. Most chunks are 3–8 KB.

Content includes decision frameworks, comparison tables, configuration guidance, anti-patterns, probing questions, and gotchas. The tone matches the agents — practical, specific, opinionated where warranted.

## Agents With Reference Directories

28 of the 31 agents have knowledge directories here. The 3 agents without reference directories (the CI/CD specialists) carry their domain knowledge entirely in their `.agent.md` prompt instructions.

---

## Full Table of Contents

### azure-ai-specialist/ (7 files)

| File | Topic |
|------|-------|
| [_toc.md](azure-ai-specialist/_toc.md) | Knowledge index |
| [agent-frameworks.md](azure-ai-specialist/agent-frameworks.md) | MAF, Semantic Kernel, Prompt Flow, AutoGen — framework selection |
| [ai-service-selection.md](azure-ai-specialist/ai-service-selection.md) | Azure OpenAI vs AI Foundry vs Cognitive Services vs custom models |
| [authentication-questions.md](azure-ai-specialist/authentication-questions.md) | AI service auth patterns, probing questions for architectural gaps |
| [deployment-types-economics.md](azure-ai-specialist/deployment-types-economics.md) | PTU vs pay-as-you-go, provisioning, capacity planning, token economics |
| [model-selection.md](azure-ai-specialist/model-selection.md) | GPT-4o vs GPT-4 vs GPT-3.5 vs open-source model selection |
| [rag-architecture.md](azure-ai-specialist/rag-architecture.md) | RAG design — vector stores, AI Search, chunking strategies |
| [responsible-ai.md](azure-ai-specialist/responsible-ai.md) | Content filtering, red teaming, prompt injection defense |

### azure-apps-infra-architect/ (7 files)

| File | Topic |
|------|-------|
| [_toc.md](azure-apps-infra-architect/_toc.md) | Knowledge index |
| [anti-patterns-questions.md](azure-apps-infra-architect/anti-patterns-questions.md) | Common deployment anti-patterns, probing questions |
| [authentication-identity.md](azure-apps-infra-architect/authentication-identity.md) | App registrations, managed identity, service principals, auth flows |
| [compute-selection.md](azure-apps-infra-architect/compute-selection.md) | App Service vs Container Apps vs AKS vs Functions selection |
| [deployment-patterns.md](azure-apps-infra-architect/deployment-patterns.md) | Blue-green, canary, slot swaps, IaC deployment strategies |
| [service-affinity.md](azure-apps-infra-architect/service-affinity.md) | Which Azure services work well together, pairing guidance |
| [waf-cost-operations.md](azure-apps-infra-architect/waf-cost-operations.md) | Well-Architected Framework — cost optimization, operational excellence |
| [waf-reliability-security.md](azure-apps-infra-architect/waf-reliability-security.md) | Well-Architected Framework — reliability, security pillars |

### azure-compute-engineer/ (9 files)

| File | Topic |
|------|-------|
| [_toc.md](azure-compute-engineer/_toc.md) | Knowledge index |
| [aks-deep-dive.md](azure-compute-engineer/aks-deep-dive.md) | AKS node pools, networking models, upgrade paths |
| [app-service.md](azure-compute-engineer/app-service.md) | App Service tiers, limits, deployment slots, scaling |
| [autoscaling.md](azure-compute-engineer/autoscaling.md) | VMSS, App Service, KEDA, cluster autoscaler, HPA |
| [azure-functions.md](azure-compute-engineer/azure-functions.md) | Consumption vs Premium vs Dedicated, cold starts, execution limits |
| [batch-anti-patterns.md](azure-compute-engineer/batch-anti-patterns.md) | Azure Batch, HPC workloads, compute anti-patterns |
| [compute-selection.md](azure-compute-engineer/compute-selection.md) | VM vs App Service vs Container Apps vs AKS vs Functions decision tree |
| [container-apps-aks.md](azure-compute-engineer/container-apps-aks.md) | Container Apps vs AKS — when each is appropriate |
| [cost-optimization.md](azure-compute-engineer/cost-optimization.md) | Spot instances, reserved capacity, right-sizing, scale-to-zero |
| [vm-families.md](azure-compute-engineer/vm-families.md) | A/B/D/E/F/L/M/N series — sizing, use cases, pricing |

### azure-data-engineer/ (8 files)

| File | Topic |
|------|-------|
| [_toc.md](azure-data-engineer/_toc.md) | Knowledge index |
| [cost-optimization.md](azure-data-engineer/cost-optimization.md) | Data platform cost traps, optimization strategies |
| [data-governance.md](azure-data-engineer/data-governance.md) | Purview, Unity Catalog, data quality, lineage |
| [delta-lake-optimization.md](azure-data-engineer/delta-lake-optimization.md) | Parquet, partitioning, Z-order, file compaction, query tuning |
| [ingestion-patterns.md](azure-data-engineer/ingestion-patterns.md) | Data Factory, Synapse pipelines, Fabric dataflows, streaming |
| [medallion-architecture.md](azure-data-engineer/medallion-architecture.md) | Bronze/silver/gold layers, lakehouse design patterns |
| [platform-selection.md](azure-data-engineer/platform-selection.md) | Fabric vs Synapse vs Databricks — selection criteria |
| [questions.md](azure-data-engineer/questions.md) | Probing questions for data platform engagements |
| [security.md](azure-data-engineer/security.md) | Row-level, column-level, workspace-level security |

### azure-database-specialist/ (11 files)

| File | Topic |
|------|-------|
| [_toc.md](azure-database-specialist/_toc.md) | Knowledge index |
| [azure-sql-database.md](azure-database-specialist/azure-sql-database.md) | SQL Database tiers, DTU vs vCore, elastic pools, serverless |
| [backup-restore.md](azure-database-specialist/backup-restore.md) | PITR, long-term retention, geo-restore |
| [cosmos-db.md](azure-database-specialist/cosmos-db.md) | Consistency levels, partition key design, RU math |
| [cost-optimization.md](azure-database-specialist/cost-optimization.md) | Database cost drivers and optimization patterns |
| [data-store-selection.md](azure-database-specialist/data-store-selection.md) | Relational vs NoSQL vs cache vs blob decision framework |
| [migration.md](azure-database-specialist/migration.md) | DMS, offline vs online, schema conversion, cutover |
| [performance.md](azure-database-specialist/performance.md) | Indexing, query tuning, wait stats, Intelligent Insights |
| [postgresql-mysql.md](azure-database-specialist/postgresql-mysql.md) | PostgreSQL and MySQL Flexible Servers — HA, migration |
| [redis-cache.md](azure-database-specialist/redis-cache.md) | Redis tiers, clustering, persistence, eviction |
| [replication-dr.md](azure-database-specialist/replication-dr.md) | Geo-replication, read replicas, failover groups, RTO/RPO |
| [security.md](azure-database-specialist/security.md) | TDE, Always Encrypted, DDM, RLS, Entra auth, Ledger |

### azure-integration-architect/ (9 files)

| File | Topic |
|------|-------|
| [_toc.md](azure-integration-architect/_toc.md) | Knowledge index |
| [anti-patterns.md](azure-integration-architect/anti-patterns.md) | Common integration anti-patterns |
| [api-management.md](azure-integration-architect/api-management.md) | APIM policies, products, subscriptions, developer portal |
| [cost-model.md](azure-integration-architect/cost-model.md) | Logic Apps tiers, Service Bus pricing, APIM SKUs |
| [event-grid.md](azure-integration-architect/event-grid.md) | Event Grid topics, subscriptions, filtering, dead-lettering |
| [failure-design.md](azure-integration-architect/failure-design.md) | Retry policies, circuit breakers, compensation logic |
| [hybrid-integration.md](azure-integration-architect/hybrid-integration.md) | On-prem connectivity, legacy protocols, BizTalk migration |
| [integration-overview.md](azure-integration-architect/integration-overview.md) | Service selection — Logic Apps vs Service Bus vs Event Grid vs Functions |
| [logic-apps.md](azure-integration-architect/logic-apps.md) | Logic Apps Consumption vs Standard, connectors, workflow design |
| [service-bus.md](azure-integration-architect/service-bus.md) | Queues, topics, sessions, competing consumers, poison messages |

### azure-monitoring-engineer/ (9 files)

| File | Topic |
|------|-------|
| [_toc.md](azure-monitoring-engineer/_toc.md) | Knowledge index |
| [alerting.md](azure-monitoring-engineer/alerting.md) | Alert design — symptoms not causes, actionable alerts |
| [azure-monitor-ecosystem.md](azure-monitoring-engineer/azure-monitor-ecosystem.md) | Log Analytics, Application Insights, metrics, workbooks |
| [cost-management.md](azure-monitoring-engineer/cost-management.md) | Log retention, ingestion costs, data collection rules |
| [diagnostic-settings.md](azure-monitoring-engineer/diagnostic-settings.md) | Resource diagnostics as part of deployment, not afterthought |
| [distributed-tracing.md](azure-monitoring-engineer/distributed-tracing.md) | Correlation IDs, dependency tracking, transaction visibility |
| [kql-patterns.md](azure-monitoring-engineer/kql-patterns.md) | KQL queries that find problems, not generate noise |
| [monitoring-vs-observability.md](azure-monitoring-engineer/monitoring-vs-observability.md) | Monitoring vs observability distinction and design principles |
| [three-pillars.md](azure-monitoring-engineer/three-pillars.md) | Metrics, logs, traces — when each is appropriate |
| [visualization.md](azure-monitoring-engineer/visualization.md) | Dashboards and workbooks that answer questions |

### azure-network-engineer/ (8 files)

| File | Topic |
|------|-------|
| [_toc.md](azure-network-engineer/_toc.md) | Knowledge index |
| [dns.md](azure-network-engineer/dns.md) | Private DNS zones, conditional forwarding, split-brain DNS |
| [firewalling-waf.md](azure-network-engineer/firewalling-waf.md) | Azure Firewall vs NVAs vs Front Door WAF |
| [hybrid-connectivity.md](azure-network-engineer/hybrid-connectivity.md) | ExpressRoute, VPN Gateway, BGP peering, forced tunneling |
| [limits-questions.md](azure-network-engineer/limits-questions.md) | Subnet sizes, peering limits, gateway throughput, probing questions |
| [network-topology.md](azure-network-engineer/network-topology.md) | Hub-spoke vs Virtual WAN — topology selection |
| [nsg-asg-design.md](azure-network-engineer/nsg-asg-design.md) | NSG/ASG design for maintainability and auditability |
| [private-connectivity.md](azure-network-engineer/private-connectivity.md) | Private endpoints vs service endpoints vs VNet integration |
| [troubleshooting.md](azure-network-engineer/troubleshooting.md) | NSG flow logs, connection monitor, Network Watcher, packet capture |

### azure-specialty-lead/ (7 files)

| File | Topic |
|------|-------|
| [_toc.md](azure-specialty-lead/_toc.md) | Knowledge index |
| [anti-patterns.md](azure-specialty-lead/anti-patterns.md) | Coordination anti-patterns — local optimization, silos |
| [collaboration-protocol.md](azure-specialty-lead/collaboration-protocol.md) | How to engage and coordinate multiple specialists |
| [domain-interconnections.md](azure-specialty-lead/domain-interconnections.md) | How domains affect each other — data→AI, compute→cost, network→everything |
| [solution-design-framework.md](azure-specialty-lead/solution-design-framework.md) | End-to-end solution design across domains |
| [technology-decision-trees.md](azure-specialty-lead/technology-decision-trees.md) | Cross-domain technology selection guidance |
| [well-architected-framework.md](azure-specialty-lead/well-architected-framework.md) | WAF as applied across multiple specialties |
| [work-sequencing.md](azure-specialty-lead/work-sequencing.md) | Network before compute, database before integration, monitoring from day one |

### azure-storage-engineer/ (10 files)

| File | Topic |
|------|-------|
| [_toc.md](azure-storage-engineer/_toc.md) | Knowledge index |
| [adls-gen2.md](azure-storage-engineer/adls-gen2.md) | ADLS Gen2 — hierarchical namespace, ACLs, data lake patterns |
| [anti-patterns.md](azure-storage-engineer/anti-patterns.md) | Storage anti-patterns and common mistakes |
| [azure-files-netapp.md](azure-storage-engineer/azure-files-netapp.md) | Azure Files vs NetApp Files — SMB/NFS file shares |
| [blob-access-tiers.md](azure-storage-engineer/blob-access-tiers.md) | Hot, cool, cold, archive — access tier cost math |
| [data-protection.md](azure-storage-engineer/data-protection.md) | Soft delete, versioning, point-in-time restore |
| [lifecycle-management.md](azure-storage-engineer/lifecycle-management.md) | Lifecycle policies and tier transitions |
| [performance.md](azure-storage-engineer/performance.md) | IOPS limits, throughput limits, performance tuning |
| [redundancy.md](azure-storage-engineer/redundancy.md) | LRS, ZRS, GRS, GZRS — replication options and tradeoffs |
| [security.md](azure-storage-engineer/security.md) | SAS tokens, managed identity, private endpoints, CMK, immutable storage |
| [storage-selection.md](azure-storage-engineer/storage-selection.md) | Blob vs ADLS vs Files vs NetApp selection framework |

### azure-terraform-author/ (8 files)

| File | Topic |
|------|-------|
| [_toc.md](azure-terraform-author/_toc.md) | Knowledge index |
| [code-patterns.md](azure-terraform-author/code-patterns.md) | Resource patterns, data sources, dynamic blocks, for_each |
| [code-quality.md](azure-terraform-author/code-quality.md) | tflint, checkov, naming conventions, code review standards |
| [common-gotchas.md](azure-terraform-author/common-gotchas.md) | Provider bugs, deprecations, breaking changes, workarounds |
| [managed-identity.md](azure-terraform-author/managed-identity.md) | Managed identity patterns in Terraform — assignments, role bindings |
| [module-architecture.md](azure-terraform-author/module-architecture.md) | Module design, AVM, registry, composition patterns |
| [provider-landscape.md](azure-terraform-author/provider-landscape.md) | azurerm vs azapi, provider versioning, feature gaps |
| [state-management.md](azure-terraform-author/state-management.md) | Backends, workspaces, state locking, migration |
| [testing.md](azure-terraform-author/testing.md) | terraform test, terratest, plan validation strategies |

### bicep-code-author/ (8 files)

| File | Topic |
|------|-------|
| [_toc.md](bicep-code-author/_toc.md) | Knowledge index |
| [best-practices.md](bicep-code-author/best-practices.md) | Naming, secure parameters, Key Vault references, linter compliance |
| [deployment-scopes.md](bicep-code-author/deployment-scopes.md) | Resource group, subscription, management group, tenant scopes |
| [deployment-stacks.md](bicep-code-author/deployment-stacks.md) | Deployment stacks — lifecycle management, deny-delete, purge |
| [language-fundamentals.md](bicep-code-author/language-fundamentals.md) | Resource declarations, decorators, user-defined types, imports |
| [modules-avm.md](bicep-code-author/modules-avm.md) | Module patterns, Azure Verified Modules, registry, composition |
| [parameter-files.md](bicep-code-author/parameter-files.md) | .bicepparam format, compile-time vs deploy-time configuration |
| [state-deployment-model.md](bicep-code-author/state-deployment-model.md) | Declarative idempotent model, deployment history, Terraform comparison |
| [what-if-linter.md](bicep-code-author/what-if-linter.md) | What-if validation, linter rules, preflight checks |

### caf-evangelist/ (7 files)

| File | Topic |
|------|-------|
| [_toc.md](caf-evangelist/_toc.md) | Knowledge index |
| [caf-phases.md](caf-evangelist/caf-phases.md) | The 7 CAF phases — Strategy, Plan, Ready, Adopt, Govern, Secure, Manage |
| [caf-reality-check.md](caf-evangelist/caf-reality-check.md) | Where CAF is right, where it's overkill, where implementations diverge |
| [governance-disciplines.md](caf-evangelist/governance-disciplines.md) | Cost, security baseline, identity, resource consistency, deployment acceleration |
| [landing-zone-architecture.md](caf-evangelist/landing-zone-architecture.md) | CAF landing zone reference architectures |
| [maturity-model.md](caf-evangelist/maturity-model.md) | Organizational cloud maturity assessment |
| [migration-strategies.md](caf-evangelist/migration-strategies.md) | The 8 Rs of migration — rehost through retire |
| [stakeholder-buy-in.md](caf-evangelist/stakeholder-buy-in.md) | Getting buy-in from security, finance, ops, dev teams |

### cloud-engineering-org/ (6 files)

| File | Topic |
|------|-------|
| [_toc.md](cloud-engineering-org/_toc.md) | Knowledge index |
| [communication-model.md](cloud-engineering-org/communication-model.md) | Executive, engineer, and PM communication styles |
| [dependency-management.md](cloud-engineering-org/dependency-management.md) | Cross-division dependencies, parallelization, handoffs |
| [engagement-lifecycle.md](cloud-engineering-org/engagement-lifecycle.md) | Scoping, assessment, foundation, build, validate, handoff phases |
| [engagement-patterns.md](cloud-engineering-org/engagement-patterns.md) | Migration, new app, M365 modernization, cost control patterns |
| [mandatory-gates.md](cloud-engineering-org/mandatory-gates.md) | Security veto, cost approval, documentation gates |
| [organization-structure.md](cloud-engineering-org/organization-structure.md) | 4 divisions, shared services, agent roster |

### cost-optimization-specialist/ (8 files)

| File | Topic |
|------|-------|
| [_toc.md](cost-optimization-specialist/_toc.md) | Knowledge index |
| [azure-cost-management.md](cost-optimization-specialist/azure-cost-management.md) | Cost Management portal, Advisor recommendations, cost exports |
| [budgets-alerts.md](cost-optimization-specialist/budgets-alerts.md) | Budget configuration, alert thresholds, action groups |
| [commitment-discounts.md](cost-optimization-specialist/commitment-discounts.md) | Reserved instances vs savings plans vs spot — modeling and selection |
| [cost-drivers.md](cost-optimization-specialist/cost-drivers.md) | Per-service cost drivers — Cosmos RU, storage transactions, egress |
| [cost-review-reporting.md](cost-optimization-specialist/cost-review-reporting.md) | Cost reports for executives and engineers |
| [finding-waste.md](cost-optimization-specialist/finding-waste.md) | Common waste patterns — orphaned resources, oversized SKUs |
| [finops-framework.md](cost-optimization-specialist/finops-framework.md) | FinOps as continuous practice, maturity model |
| [tagging-strategy.md](cost-optimization-specialist/tagging-strategy.md) | Tag design for cost allocation and chargeback |

### devops-lead/ (8 files)

| File | Topic |
|------|-------|
| [_toc.md](devops-lead/_toc.md) | Knowledge index |
| [ado-github-migration.md](devops-lead/ado-github-migration.md) | ADO to GitHub migration planning and execution |
| [azure-pipelines-templates.md](devops-lead/azure-pipelines-templates.md) | ADO pipeline template architecture at scale |
| [devops-maturity.md](devops-lead/devops-maturity.md) | Maturity assessment and improvement prescriptions |
| [dora-metrics.md](devops-lead/dora-metrics.md) | Deployment frequency, lead time, change failure rate, MTTR |
| [github-actions-components.md](devops-lead/github-actions-components.md) | Reusable workflows, composite actions, matrix strategies |
| [golden-paths.md](devops-lead/golden-paths.md) | Paved roads, golden paths, developer experience |
| [innersource.md](devops-lead/innersource.md) | Sharing automation across teams without creating a monolith |
| [toolchain-selection.md](devops-lead/toolchain-selection.md) | ADO vs GitHub vs hybrid — platform selection criteria |

### documentation-writer/ (7 files)

| File | Topic |
|------|-------|
| [_toc.md](documentation-writer/_toc.md) | Knowledge index |
| [adrs.md](documentation-writer/adrs.md) | Architecture Decision Records — format, when to write them |
| [audience-adaptation.md](documentation-writer/audience-adaptation.md) | Writing for executives vs engineers vs ops vs new hires |
| [consistency-tribal-knowledge.md](documentation-writer/consistency-tribal-knowledge.md) | Naming conventions, formatting standards, capturing tribal knowledge |
| [documentation-types.md](documentation-writer/documentation-types.md) | Tutorials, how-to guides, reference docs, explanations |
| [platforms-formatting.md](documentation-writer/platforms-formatting.md) | README, wiki, Confluence, SharePoint, Docusaurus |
| [runbooks.md](documentation-writer/runbooks.md) | Operational runbook format and content |
| [writing-style.md](documentation-writer/writing-style.md) | Writing principles and style guide |

### entra-id-specialist/ (9 files)

| File | Topic |
|------|-------|
| [_toc.md](entra-id-specialist/_toc.md) | Knowledge index |
| [app-registrations-service-principals.md](entra-id-specialist/app-registrations-service-principals.md) | App registrations vs enterprise apps (service principals) |
| [authentication-protocols.md](entra-id-specialist/authentication-protocols.md) | OAuth 2.0, OIDC, SAML — when to use each |
| [b2b-b2c-external-id.md](entra-id-specialist/b2b-b2c-external-id.md) | B2B guest access, B2C/External ID customer identity |
| [conditional-access.md](entra-id-specialist/conditional-access.md) | Policy layering, break-glass accounts, lockout prevention |
| [hybrid-identity.md](entra-id-specialist/hybrid-identity.md) | Entra Connect, Cloud Sync, PHS vs PTA vs federation |
| [legacy-auth.md](entra-id-specialist/legacy-auth.md) | Killing legacy auth protocols without breaking production |
| [pim.md](entra-id-specialist/pim.md) | Just-in-time access, role activation, access reviews |
| [tenant-audit-questions.md](entra-id-specialist/tenant-audit-questions.md) | Tenant audit — stale service principals, over-permissioned apps |
| [workload-identity-federation.md](entra-id-specialist/workload-identity-federation.md) | GitHub Actions, Kubernetes — secretless auth for service principals |

### identity-productivity-lead/ (7 files)

| File | Topic |
|------|-------|
| [_toc.md](identity-productivity-lead/_toc.md) | Knowledge index |
| [coordination-model.md](identity-productivity-lead/coordination-model.md) | Coordinating Entra ID, Teams, and M365 specialists |
| [cross-service-policies.md](identity-productivity-lead/cross-service-policies.md) | Conditional access, Purview, Intune — unified enforcement |
| [identity-foundation.md](identity-productivity-lead/identity-foundation.md) | Identity as the foundation everything else depends on |
| [licensing-optimization.md](identity-productivity-lead/licensing-optimization.md) | E3 vs E5, bundled vs add-on, SKU optimization |
| [organizational-challenges.md](identity-productivity-lead/organizational-challenges.md) | Shadow IT, user adoption, change management |
| [security-usability-balance.md](identity-productivity-lead/security-usability-balance.md) | Balancing security with usability — the most secure unused system isn't secure |
| [unified-platform-view.md](identity-productivity-lead/unified-platform-view.md) | M365 as integrated platform, not a pile of apps |

### landing-zone-specialist/ (8 files)

| File | Topic |
|------|-------|
| [_toc.md](landing-zone-specialist/_toc.md) | Knowledge index |
| [architecture-overview.md](landing-zone-specialist/architecture-overview.md) | ALZ conceptual architecture, management group hierarchy |
| [greenfield-brownfield.md](landing-zone-specialist/greenfield-brownfield.md) | Greenfield deployment vs brownfield migration |
| [identity-at-scale.md](landing-zone-specialist/identity-at-scale.md) | PIM, conditional access, break-glass accounts, SP hygiene |
| [implementation-options.md](landing-zone-specialist/implementation-options.md) | ALZ accelerator — Bicep vs Terraform vs Portal |
| [network-topology.md](landing-zone-specialist/network-topology.md) | Hub-spoke vs Virtual WAN, topology migration |
| [policy-governance.md](landing-zone-specialist/policy-governance.md) | Policy-driven governance — enforce, audit, exempt, DINE |
| [questions.md](landing-zone-specialist/questions.md) | Probing questions for landing zone engagements |
| [subscription-vending.md](landing-zone-specialist/subscription-vending.md) | Subscription vending automation and guardrails |

### m365-engineer/ (9 files)

| File | Topic |
|------|-------|
| [_toc.md](m365-engineer/_toc.md) | Knowledge index |
| [exchange-online.md](m365-engineer/exchange-online.md) | Mail flow, connectors, transport rules, SPF/DKIM/DMARC |
| [intune.md](m365-engineer/intune.md) | Compliance policies, configuration profiles, app deployment |
| [licensing.md](m365-engineer/licensing.md) | E3 vs E5, feature-to-SKU mapping, cost optimization |
| [migration.md](m365-engineer/migration.md) | On-prem Exchange, Google Workspace, tenant-to-tenant migration |
| [platform-overview.md](m365-engineer/platform-overview.md) | Full M365 stack overview and integration points |
| [power-platform-governance.md](m365-engineer/power-platform-governance.md) | Environments, DLP policies, CoE toolkit, citizen dev governance |
| [purview.md](m365-engineer/purview.md) | Sensitivity labels, retention, DLP, insider risk |
| [sharepoint-online.md](m365-engineer/sharepoint-online.md) | Site architecture, hub sites, content types, search |
| [tenant-architecture.md](m365-engineer/tenant-architecture.md) | Single vs multi-tenant, geo, sovereign clouds |

### platform-engineering-lead/ (9 files)

| File | Topic |
|------|-------|
| [_toc.md](platform-engineering-lead/_toc.md) | Knowledge index |
| [build-buy-adopt.md](platform-engineering-lead/build-buy-adopt.md) | Build vs buy vs adopt decision framework |
| [capability-model.md](platform-engineering-lead/capability-model.md) | Platform Engineering Capability Model assessment |
| [cloud-operating-models.md](platform-engineering-lead/cloud-operating-models.md) | Centralized, federated, and hybrid operating models |
| [fundamentals.md](platform-engineering-lead/fundamentals.md) | What platform engineering is and why it matters |
| [landing-zone-integration.md](platform-engineering-lead/landing-zone-integration.md) | Ensuring landing zones get used and governance gets followed |
| [paved-paths.md](platform-engineering-lead/paved-paths.md) | Golden paths, paved roads, developer self-service |
| [roadmap-maturity.md](platform-engineering-lead/roadmap-maturity.md) | 6-month roadmaps, maturity progression, tech debt management |
| [stakeholder-translation.md](platform-engineering-lead/stakeholder-translation.md) | Translating between executives and engineers |
| [team-topologies.md](platform-engineering-lead/team-topologies.md) | Stream-aligned, platform, enabling, and complicated-subsystem teams |

### power-platform-engineer/ (9 files)

| File | Topic |
|------|-------|
| [_toc.md](power-platform-engineer/_toc.md) | Knowledge index |
| [alm.md](power-platform-engineer/alm.md) | Solutions, environment variables, connection references, pipelines |
| [dataverse.md](power-platform-engineer/dataverse.md) | Tables, relationships, security roles, solutions |
| [decision-guidance.md](power-platform-engineer/decision-guidance.md) | Power Platform vs custom app — when each is the right answer |
| [governance.md](power-platform-engineer/governance.md) | Environments, DLP policies, CoE toolkit, citizen dev enablement |
| [integration.md](power-platform-engineer/integration.md) | Custom connectors, Azure Functions, on-prem data gateway |
| [licensing.md](power-platform-engineer/licensing.md) | Premium vs standard connector trap, per-user vs per-app |
| [power-apps.md](power-platform-engineer/power-apps.md) | Canvas vs model-driven, custom pages, component libraries |
| [power-automate.md](power-platform-engineer/power-automate.md) | Cloud vs desktop flows, error handling, silent failures |
| [power-bi.md](power-platform-engineer/power-bi.md) | Data modeling, DAX, RLS, deployment pipelines |

### powershell-automation-dev/ (8 files)

| File | Topic |
|------|-------|
| [_toc.md](powershell-automation-dev/_toc.md) | Knowledge index |
| [authentication-patterns.md](powershell-automation-dev/authentication-patterns.md) | Managed identity, workload identity, certificate auth, interactive |
| [error-handling.md](powershell-automation-dev/error-handling.md) | Proper error handling, logging, idempotency patterns |
| [execution-environments.md](powershell-automation-dev/execution-environments.md) | Azure Automation vs Functions vs GitHub Actions vs local tasks |
| [module-development.md](powershell-automation-dev/module-development.md) | When script→function→module, private gallery publishing |
| [module-ecosystem.md](powershell-automation-dev/module-ecosystem.md) | Graph SDK, Az modules, EXO, Teams, PnP — when to use each |
| [performance-optimization.md](powershell-automation-dev/performance-optimization.md) | ForEach-Object -Parallel, pipeline optimization, throttling |
| [pester-testing.md](powershell-automation-dev/pester-testing.md) | Pester test patterns, mocking, code coverage |
| [ps7-language-features.md](powershell-automation-dev/ps7-language-features.md) | PowerShell 7 features — ternary, null-coalescing, SSH remoting |

### retrospective-agent/ (8 files)

| File | Topic |
|------|-------|
| [_toc.md](retrospective-agent/_toc.md) | Knowledge index |
| [blameless-culture.md](retrospective-agent/blameless-culture.md) | Blameless retrospective principles and facilitation |
| [common-patterns.md](retrospective-agent/common-patterns.md) | Recurring failure patterns across engagements |
| [dora-metrics.md](retrospective-agent/dora-metrics.md) | DORA metrics as retrospective inputs |
| [improvement-tracking.md](retrospective-agent/improvement-tracking.md) | Action item tracking, follow-through accountability |
| [maturity-model.md](retrospective-agent/maturity-model.md) | Retrospective maturity — from blame to systemic improvement |
| [retrospective-template.md](retrospective-agent/retrospective-template.md) | Retrospective document format with findings, actions, owners |
| [retrospective-types.md](retrospective-agent/retrospective-types.md) | Sprint, engagement, incident, and architecture retrospective types |
| [root-cause-analysis.md](retrospective-agent/root-cause-analysis.md) | 5 Whys, fishbone diagrams, contributing factors analysis |

### security-compliance-analyst/ (9 files)

| File | Topic |
|------|-------|
| [_toc.md](security-compliance-analyst/_toc.md) | Knowledge index |
| [compliance-frameworks.md](security-compliance-analyst/compliance-frameworks.md) | SOC2, HIPAA, PCI-DSS, FedRAMP — requirements vs auditor expectations |
| [data-protection.md](security-compliance-analyst/data-protection.md) | PII handling, encryption at rest and in transit, retention |
| [defender-for-cloud.md](security-compliance-analyst/defender-for-cloud.md) | Defender for Cloud posture management and recommendations |
| [identity-security.md](security-compliance-analyst/identity-security.md) | Least privilege, break-glass procedures, SP scoping |
| [network-security.md](security-compliance-analyst/network-security.md) | Traffic encryption, private endpoints, public exposure |
| [security-frameworks.md](security-compliance-analyst/security-frameworks.md) | Azure Security Benchmark, CIS Controls, NIST mapping |
| [security-review-template.md](security-compliance-analyst/security-review-template.md) | Security review document format for auditor consumption |
| [sentinel.md](security-compliance-analyst/sentinel.md) | Microsoft Sentinel — SIEM/SOAR, analytics rules, playbooks |
| [threat-modeling.md](security-compliance-analyst/threat-modeling.md) | STRIDE, attack surface identification, remediation proposals |

### teams-admin/ (9 files)

| File | Topic |
|------|-------|
| [_toc.md](teams-admin/_toc.md) | Knowledge index |
| [admin-centers-policies.md](teams-admin/admin-centers-policies.md) | Teams Admin Center, policy types, inheritance model |
| [call-quality.md](teams-admin/call-quality.md) | CQD, Call Analytics, network assessment, troubleshooting |
| [call-routing-aa-cq.md](teams-admin/call-routing-aa-cq.md) | Auto attendants, call queues, resource accounts |
| [compliance.md](teams-admin/compliance.md) | Retention, eDiscovery, communication compliance, DLP |
| [guest-external-access.md](teams-admin/guest-external-access.md) | Guest access model, external federation, tenant exposure |
| [phone-system.md](teams-admin/phone-system.md) | Direct Routing vs Operator Connect vs Calling Plans, SBCs |
| [questions.md](teams-admin/questions.md) | Probing questions for Teams administration engagements |
| [rooms-devices.md](teams-admin/rooms-devices.md) | Teams Rooms provisioning, monitoring, firmware, resource accounts |
| [team-lifecycle.md](teams-admin/team-lifecycle.md) | Creation policies, expiration, archival, naming conventions |

### testing-validation-engineer/ (9 files)

| File | Topic |
|------|-------|
| [_toc.md](testing-validation-engineer/_toc.md) | Knowledge index |
| [application-testing.md](testing-validation-engineer/application-testing.md) | Unit, integration, e2e testing — and why teams skip them |
| [azure-load-testing.md](testing-validation-engineer/azure-load-testing.md) | Performance validation, finding breaking points |
| [bicep-testing.md](testing-validation-engineer/bicep-testing.md) | What-if, ARM validation, deployment preflight |
| [compliance-drift.md](testing-validation-engineer/compliance-drift.md) | Policy compliance validation, regulatory dashboard accuracy |
| [devops-test-principles.md](testing-validation-engineer/devops-test-principles.md) | Testing as discipline not checkbox, testing theater vs real testing |
| [pipeline-validation.md](testing-validation-engineer/pipeline-validation.md) | Does CI/CD actually do what the team thinks? |
| [powershell-testing.md](testing-validation-engineer/powershell-testing.md) | Pester patterns for infrastructure validation |
| [terraform-testing.md](testing-validation-engineer/terraform-testing.md) | terraform test, terratest, plan validation, checkov, tflint |
| [test-strategy.md](testing-validation-engineer/test-strategy.md) | Designing maintainable test strategies |

---

## Stats

- **28** agent knowledge directories
- **28** `_toc.md` index files
- **231** topic chunk files
- **259** total files

> **Last Updated:** 2026-04-19 — Knowledge refined with anti-patterns, gotchas, best-practices, AVM stance update, and gap analysis across all 31 reference directories.
