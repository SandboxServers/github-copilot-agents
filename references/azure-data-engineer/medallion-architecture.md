# Medallion Architecture

Three-layer data quality architecture for lakehouse platforms.

## Bronze Layer (Raw)

- Ingest data exactly as received from source systems — no transformations
- Append-only writes to preserve full fidelity and audit trail
- Schema-on-read: store in source format converted to Delta tables
- Partition by ingestion date for lifecycle management
- Long retention — serves as the source of truth for reprocessing
- No data quality rules applied — raw is raw
- Include metadata columns: source system, ingestion timestamp, batch ID
- Format conversion (CSV/JSON → Delta) is acceptable; data modification is not

## Silver Layer (Cleansed)

- Deduplicated records using business keys or composite keys
- Validated against expected schemas with type casting and null handling
- Standardized naming conventions and conformed dimensions
- Business keys resolved across source systems (customer ID mapping)
- Incremental processing using watermark columns or change tracking
- Cross-source integration and joins happen at this layer
- Quality gates: schema enforcement, referential integrity checks
- Managed Delta tables with enforced schemas
- Partition by business keys (date, region, entity type)

## Gold Layer (Curated)

- Business-level aggregations optimized for specific consumers
- Star schema or denormalized models for BI consumption
- Pre-computed metrics and KPIs for dashboard performance
- Materialized views for frequently accessed aggregations
- Purpose-built tables — different gold tables for different use cases
- Z-ordered on common query columns for fast reads
- Row-level and column-level security applied for data access control
- Ready for Power BI DirectLake, ML feature stores, or API serving
- Business rules enforced with defined SLAs for freshness

## Data Flow

```
Source Systems → Landing Zone (optional, raw files)
  → Bronze (raw ingestion, append-only Delta)
    → Silver (quality + conformance + deduplication)
      → Gold (business aggregation + dimensional models)
        → Consumption (BI / ML / APIs / External partners)
```

## Implementation in Fabric

- Each layer maps to a Fabric Lakehouse or Warehouse
- Bronze: Lakehouse with raw Delta tables, loaded via pipelines or Dataflow Gen2
- Silver: Lakehouse with managed tables, processed via Spark notebooks
- Gold: Warehouse or Lakehouse with SQL endpoint for BI access
- Shortcuts for cross-lakehouse references without data duplication
- OneLake stores all layers in open Delta Parquet format

## Implementation in Databricks

- Unity Catalog organizes layers as catalogs or schemas
- Bronze: raw catalog with append-only tables, loaded via Auto Loader
- Silver: cleansed catalog with Delta Live Tables for quality expectations
- Gold: curated catalog with Z-ordered, optimized tables
- Delta Live Tables automate dependency management across layers
- Time travel available at every layer for auditing and replay

## Delta Tables at Every Layer

- ACID transactions ensure consistency during concurrent reads/writes
- Time travel enables auditing and rollback to previous versions
- Schema enforcement prevents bad data from corrupting tables
- Schema evolution supports adding columns without breaking consumers
- OPTIMIZE compacts small files at each layer for query performance

## Common Mistakes

- Gold is just Silver with a rename — gold should be purpose-built for consumers
- No quality gates between layers — data quality must increase at each transition
- Bronze isn't truly raw — don't filter or modify data in bronze
- Everything in one lakehouse — separate by domain; each team owns their chain
- No replay capability — bronze must support full reprocessing from scratch
- Skipping silver — going bronze-to-gold creates unmaintainable pipelines
- Not tracking lineage — use Purview or Unity Catalog to trace data flow
