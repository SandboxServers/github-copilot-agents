# Platform Selection

## Platform Selection Decision Tree

```
What's the data platform need?
├─ End-to-end analytics SaaS (unified experience)?
│  └─ Microsoft Fabric
│     ├─ Best when: new greenfield, Power BI-centric org, want SaaS simplicity
│     ├─ Includes: Data Factory, Spark, SQL warehouse, Real-Time Intelligence, Power BI
│     ├─ Storage: OneLake (unified, open Delta Parquet format)
│     └─ Governance: Built-in + Microsoft Purview integration
├─ Advanced Spark / ML workloads with fine-grained control?
│  └─ Azure Databricks
│     ├─ Best when: heavy data science, complex ETL, Unity Catalog governance
│     ├─ Includes: Spark, MLflow, SQL warehouses, Delta Live Tables, Unity Catalog
│     ├─ Storage: ADLS Gen2 (Delta Lake format)
│     └─ Governance: Unity Catalog (native) + Purview integration
├─ Enterprise DW with T-SQL focus?
│  └─ Azure Synapse Analytics (dedicated SQL pools)
│     ├─ Best when: existing SQL Server expertise, massive parallel DW workloads
│     ├─ CAUTION: Synapse Spark is being superseded by Fabric Spark
│     └─ COST TRAP: Dedicated pools bill 24/7 unless paused
├─ Simple relational analytics?
│  └─ Azure SQL Database
│     ├─ Best when: <1TB data, relational model fits, team knows SQL
│     └─ Don't over-engineer — sometimes a SQL DB is the right answer
├─ Real-time streaming analytics?
│  ├─ Fabric Real-Time Intelligence (Event Streams + KQL)
│  ├─ Azure Databricks Structured Streaming
│  └─ Azure Stream Analytics (simpler, SQL-like)
└─ Need multiple platforms?
   └─ Common pattern: Databricks for engineering + Fabric for BI serving
      Or: Fabric for everything + Databricks for advanced ML only
```

## Platform Comparison Matrix

| Factor | Microsoft Fabric | Azure Databricks | Synapse Analytics |
|--------|-----------------|------------------|-------------------|
| Pricing model | Capacity-based (CU) | DBU-based (per cluster) | DWU (dedicated) or per-query (serverless) |
| Spark support | Yes (3.4, 3.5, 4.0) | Yes (latest, GPU support) | Yes (but migrating to Fabric) |
| SQL engine | Warehouse + SQL endpoint | Photon SQL warehouse | Dedicated/Serverless SQL pools |
| Real-time | Event Streams + KQL | Structured Streaming | Stream Analytics integration |
| Governance | Built-in + Purview | Unity Catalog + Purview | Limited (Purview scan) |
| Storage | OneLake (managed) | ADLS Gen2 (customer-managed) | ADLS Gen2 + managed storage |
| CI/CD | Git integration + deployment pipelines | Repos + Databricks Asset Bundles | Limited |
| GPU support | No | Yes | No |
| .NET Spark | No | No | Yes (legacy) |
| Scale to zero | Yes (pause capacity) | Yes (terminate clusters) | Serverless only |
| Best for | Unified analytics + BI | Complex engineering + ML | Legacy DW migrations |
