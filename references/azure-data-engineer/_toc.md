# Azure Data Engineer — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Platform Selection | [platform-selection.md](platform-selection.md) | When choosing between Fabric, Databricks, Synapse, or Data Factory |
| Medallion Architecture | [medallion-architecture.md](medallion-architecture.md) | When designing bronze/silver/gold data layers or lakehouse patterns |
| Ingestion Patterns | [ingestion-patterns.md](ingestion-patterns.md) | When designing ETL/ELT pipelines, batch vs streaming, or data movement |
| Cost Optimization | [cost-optimization.md](cost-optimization.md) | When managing data platform costs — Fabric capacity, Databricks DBUs, compute sizing |
| Data Governance | [data-governance.md](data-governance.md) | When implementing Purview, Unity Catalog, data lineage, or data quality |
| Security | [security.md](security.md) | When securing data platforms — network isolation, access control, encryption |
| Delta Lake Optimization | [delta-lake-optimization.md](delta-lake-optimization.md) | When tuning Delta Lake performance — compaction, Z-ordering, vacuum, caching |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing designs for common data engineering mistakes to avoid |
| Gotchas | [gotchas.md](gotchas.md) | When hitting unexpected Fabric, Delta Lake, ADF, or Synapse behavior |
| Best Practices | [best-practices.md](best-practices.md) | When building pipelines — medallion, incremental loads, quality, governance, monitoring |
| Real-Time Analytics | [real-time-analytics.md](real-time-analytics.md) | When designing streaming pipelines — Eventhouse, KQL, Eventstream, data mesh |
| Questions | [questions.md](questions.md) | When probing for data platform requirements or identifying architecture gaps |
