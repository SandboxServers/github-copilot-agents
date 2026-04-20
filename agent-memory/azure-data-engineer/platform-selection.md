# Data Platform Selection

Decision matrix for choosing the right Azure data platform.

## Microsoft Fabric

- Unified analytics platform: lakehouse, warehouse, real-time analytics, data science, Power BI
- OneLake as single storage layer with open Delta Parquet format
- Power BI native integration — seamless from ingestion to reporting
- Best for Microsoft-centric organizations starting new greenfield projects
- Capacity-based pricing with F SKUs (F2 at ~$262/month for dev)
- Includes Data Factory pipelines, Spark notebooks, SQL warehouse, Event Streams
- Built-in governance with Purview integration and sensitivity labels
- SaaS model — Microsoft manages infrastructure, patching, scaling

## Azure Synapse Analytics

- Dedicated SQL pools (formerly Azure SQL Data Warehouse) for massive parallel processing
- Serverless SQL pools for ad-hoc queries over data lake files
- Spark pools for big data processing (being superseded by Fabric Spark)
- Being partially superseded by Fabric — still valid for existing deployments
- Cost trap: dedicated pools bill 24/7 unless explicitly paused
- Best for organizations with heavy T-SQL investment and existing Synapse workloads
- Consider migration path to Fabric for new development

## Azure Databricks

- Best-in-class Apache Spark implementation with Photon acceleration
- Unity Catalog for centralized governance across workspaces
- MLflow for end-to-end MLOps: experiment tracking, model registry, serving
- Delta Live Tables for declarative ETL with quality expectations
- Best for teams with Spark expertise, multi-cloud requirements, or heavy ML workloads
- GPU support for deep learning and large model training
- DBU-based pricing — consumption varies by cluster type and size
- Databricks Asset Bundles for CI/CD and infrastructure-as-code

## Azure Data Factory

- Orchestration and ETL service with 100+ native connectors
- Mapping data flows for Spark-based visual transformations at scale
- Copy activity for simple data movement between stores
- Use when Fabric notebooks aren't sufficient or need complex scheduling
- Integration runtimes: Azure (cloud), self-hosted (on-prem), SSIS
- Event-triggered and schedule-based pipeline execution
- Metadata-driven patterns for parameterized ingestion at scale

## Platform Comparison

| Factor | Microsoft Fabric | Azure Databricks | Synapse Analytics | Data Factory |
|--------|-----------------|------------------|-------------------|--------------|
| Best for | Unified analytics + BI | Complex engineering + ML | Legacy DW migrations | Orchestration + ETL |
| Governance | Built-in + Purview | Unity Catalog + Purview | Purview scan | Purview lineage |
| Spark | Yes (managed) | Yes (best-in-class) | Yes (migrating) | Mapping data flows |
| SQL | Warehouse + endpoint | Photon SQL warehouse | Dedicated/Serverless pools | N/A |
| Real-time | Event Streams + KQL | Structured Streaming | Stream Analytics | Event triggers |
| Cost model | Capacity (F SKUs) | DBU per cluster | DWU or per-query | Activity runs + DIU |

## Selection Guidance

- New Microsoft-centric project with BI focus → Fabric
- Advanced Spark/ML with multi-cloud needs → Databricks
- Existing SQL DW with T-SQL expertise → Synapse (plan Fabric migration)
- Complex orchestration across many sources → Data Factory
- Common hybrid: Databricks for engineering + Fabric for BI serving
- Don't over-engineer — sometimes Azure SQL Database is the right answer for <1TB

## Key Decision Factors

- Team skills: SQL-heavy teams lean Synapse/Fabric; Python/Spark teams lean Databricks
- BI tool: Power BI users benefit most from Fabric's native integration
- Multi-cloud: Databricks runs on AWS/GCP/Azure; Fabric and Synapse are Azure-only
- ML maturity: Databricks has the strongest MLOps story with MLflow
- Budget model: Fabric is capacity-reserved; Databricks is consumption-based
- Data volume: All platforms handle TB-PB; choice depends on workload patterns
