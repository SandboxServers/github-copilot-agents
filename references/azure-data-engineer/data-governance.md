# Data Governance

## Purview + Unity Catalog Decision

```
Which governance tool?
├─ Using Databricks? → Unity Catalog (primary) + Purview (enterprise catalog)
├─ Using Fabric only? → Purview Unified Catalog (native integration)
├─ Multi-platform? → Purview as the federated catalog, Unity Catalog for Databricks-specific
└─ Legacy Synapse? → Purview scan + classification
```

## Purview Capabilities

- **Data Map**: Automated scanning of 200+ sources (Azure, AWS, GCP, on-prem)
- **Unified Catalog**: AI-powered discovery, governance domains, data products
- **Lineage**: Automated for ADF, Fabric, Databricks (with Unity Catalog connector)
- **Classification**: Auto-classification of PII, sensitive data (100+ built-in classifiers)
- **Sensitivity labels**: Inherited from Microsoft 365, applied to data assets
- **Access policies**: Self-service access requests with approval workflows
- **Data quality**: Profiling, quality rules, health monitoring

## Unity Catalog Capabilities

- **Centralized metastore**: Single catalog across all Databricks workspaces
- **Three-level namespace**: catalog.schema.table
- **Fine-grained access**: Table, column, row-level security
- **Data lineage**: Automatic lineage tracking for Spark jobs
- **Delta Sharing**: Open protocol for sharing data across platforms
- **Unity Catalog volumes**: Managed file storage with governance
- **AI/ML governance**: Model registry, feature store integration
