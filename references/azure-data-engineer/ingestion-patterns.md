# Ingestion Patterns

## Data Factory / Fabric Pipelines

| Pattern | Use Case | Key Settings |
|---------|----------|--------------|
| Full load | Small tables, reference data | Simple copy activity, overwrite |
| Incremental (watermark) | Large tables with modified dates | High watermark column, state store |
| CDC (Change Data Capture) | Source supports CDC | SQL CDC, Debezium, ADF CDC |
| Metadata-driven | Many tables, similar patterns | JSON config drives parameterized pipeline |
| Event-triggered | File arrivals, API webhooks | Storage event trigger, custom event |
| Tumbling window | Time-partitioned data | Non-overlapping time windows, retry |

## Databricks Ingestion

| Tool | Use Case | Notes |
|------|----------|-------|
| Auto Loader | File-based ingestion (cloud storage) | Exactly-once, schema inference/evolution, checkpointing |
| Lakeflow Connect | Managed connectors (databases, SaaS) | No-code, scheduled sync, built-in CDC |
| Delta Live Tables / Lakeflow Declarative | Quality-enforced pipelines | Expectations, automatic dependency management |
| Structured Streaming | Low-latency stream processing | Kafka, Event Hubs, continuous or trigger-based |

## Fabric Ingestion

| Tool | Use Case | Notes |
|------|----------|-------|
| Data pipelines | Orchestration, copy activities | ADF-compatible, 300+ connectors |
| Dataflow Gen2 | Low-code transforms (Power Query) | Good for simple reshaping, limited at scale |
| Event Streams | Real-time ingestion | Kafka-compatible, connects to KQL databases |
| Mirroring | Database replication | Near real-time sync from SQL, Cosmos, Snowflake |
| Shortcuts | Zero-copy access to external data | ADLS, S3, Dataverse — no data movement |
