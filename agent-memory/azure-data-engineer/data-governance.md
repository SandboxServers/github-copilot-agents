# Data Governance

Unified governance across data platforms using Purview, Unity Catalog, and Fabric.

## Microsoft Purview

- Unified governance service: data catalog, lineage, classification, sensitivity labels
- Data Map: automated scanning of 200+ sources (Azure, AWS, GCP, on-premises)
- Unified Catalog: AI-powered discovery, governance domains, data products
- Auto-classification of PII and sensitive data using 100+ built-in classifiers
- Sensitivity labels inherited from Microsoft 365, applied to data assets
- Self-service access requests with approval workflows
- Data quality: profiling rules, quality scoring, health monitoring dashboards
- End-to-end lineage: automatic for Data Factory, Fabric, and Databricks (via connector)
- Impact analysis: understand downstream effects before changing schemas or pipelines

## Unity Catalog (Databricks)

- Centralized metastore shared across all Databricks workspaces
- Three-level namespace: catalog.schema.table for organized data access
- Fine-grained access control: table-level, column-level, and row-level security
- Table ACLs with GRANT/REVOKE SQL syntax
- Automatic data lineage tracking for all Spark jobs and notebooks
- Delta Sharing: open protocol for sharing data across platforms and organizations
- Unity Catalog volumes: governed file storage for unstructured data
- AI/ML governance: model registry and feature store integration
- Cross-workspace governance — single policy layer across environments

## Fabric OneLake Governance

- Workspace-level security with role-based access (Admin, Member, Contributor, Viewer)
- Item-level permissions for lakehouses, warehouses, reports, notebooks
- Sensitivity labels from Microsoft 365 propagated to data items
- Data domains for organizing workspaces by business area
- Endorsement: certified and promoted labels for trusted datasets
- SQL endpoint security: row-level and column-level security via T-SQL
- OneLake data access roles for folder-level permissions within lakehouses

## Data Quality

- Great Expectations: open-source quality framework, works with Spark/Pandas
- Databricks expectations: built into Delta Live Tables with `EXPECT` constraints
- Fabric data quality rules: integrated with Lakehouse and Warehouse
- Quality metrics enforced at bronze/silver boundary as gate checks
- Quality dimensions: completeness, accuracy, consistency, timeliness, uniqueness
- Quality dashboards: track quality scores over time, alert on degradation
- Quarantine pattern: route failed records to error tables for investigation

## Lineage

- Purview tracks end-to-end lineage across Data Factory, Fabric, Databricks
- Fabric tracks lineage within workspaces (notebooks, pipelines, datasets)
- Unity Catalog tracks column-level lineage for Spark transformations
- Critical for compliance (GDPR right to erasure requires knowing where data lives)
- Impact analysis before schema changes — who and what will break

## Access Patterns

- Principle of least privilege: grant minimum permissions needed for the role
- Service principals for automated pipelines and CI/CD deployments
- Managed identity for compute resources (no credentials in code)
- Column-level security for PII: mask sensitive columns for non-privileged users
- Row-level security for multi-tenant data: filter rows by user context
- Dynamic data masking in SQL endpoints for partial visibility

## Data Classification

- PII detection: automatic scanning identifies names, SSNs, credit cards, emails
- Sensitivity labels: Confidential, Highly Confidential, General — inherited from M365
- Auto-classification rules: apply labels based on content patterns
- Retention policies: define how long data must be kept and when to delete
- Data residency: ensure data stays in required geographic regions
- Classification drives security policy — higher classification = stricter controls
