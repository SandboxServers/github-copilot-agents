# Data Ingestion Patterns

Patterns for getting data into lakehouse and warehouse platforms.

## Batch Ingestion

- Data Factory pipelines: copy activity for simple moves, mapping data flows for transformations
- Fabric pipelines: ADF-compatible engine with 300+ connectors, runs within Fabric capacity
- Databricks jobs: scheduled Spark notebooks or JAR tasks via Workflows
- Schedule-based execution: daily/hourly cron, tumbling windows for time-partitioned data
- Event-triggered execution: storage events, custom events, webhook triggers
- Full load for small tables and reference data (overwrite destination)
- Incremental load for large tables (watermark columns, change tracking, delta extraction)
- Metadata-driven pipelines: JSON config drives parameterized ingestion for many tables

## Streaming Ingestion

- Event Hubs → Fabric Real-Time Analytics (Event Streams + KQL databases)
- Event Hubs → Databricks Structured Streaming (continuous or trigger-based processing)
- Event Hubs → Azure Stream Analytics (simpler SQL-based transformations)
- IoT Hub for device telemetry → Event Hubs → processing layer
- Kafka-compatible endpoints for existing Kafka producers
- Exactly-once semantics with checkpointing in Structured Streaming
- Micro-batch vs continuous processing trade-off: latency vs cost

## Change Data Capture (CDC)

- Azure SQL CDC: enable on source tables, capture inserts/updates/deletes
- CDC → Event Hubs → Lakehouse for near-real-time replication
- Debezium on AKS: CDC for non-Azure databases (PostgreSQL, MySQL, Oracle)
- Fabric Mirroring: managed CDC for SQL Server, Azure SQL, Cosmos DB, Snowflake
- Databricks Lakeflow Connect: managed connectors with built-in CDC
- Key consideration: handle schema changes at source gracefully

## API Ingestion

- Data Factory REST connector for standard REST APIs
- Pagination handling: offset-based, cursor-based, next-link patterns
- Custom Azure Functions for complex API logic (OAuth flows, rate limiting)
- Store raw API responses in bronze layer, parse in silver
- Retry policies and idempotency for unreliable APIs
- Incremental extraction using API-specific timestamps or change tokens

## File Ingestion

- ADLS Gen2 landing zone: external systems drop files, event trigger starts pipeline
- Blob storage event → Event Grid → Azure Function → processing pipeline
- Databricks Auto Loader: exactly-once file ingestion with schema inference and evolution
- Support formats: CSV, JSON, Parquet, Avro, XML, Excel
- File validation: check row counts, expected schema, file size before processing
- Archive or delete processed files after successful ingestion

## Fabric-Specific Ingestion

- Data pipelines: copy activities with ADF-compatible orchestration
- Dataflow Gen2: low-code Power Query transforms (good for simple reshaping)
- Event Streams: Kafka-compatible real-time ingestion to KQL databases
- Mirroring: near-real-time database replication without pipelines
- Shortcuts: zero-copy access to ADLS, S3, Dataverse — no data movement needed

## Incremental Load Patterns

- Watermark columns: track max modified_date or sequence_number between runs
- Change tracking: SQL Server native change tracking for efficient delta extraction
- Delta extraction: compare source hash with existing records to detect changes
- Merge patterns: MERGE INTO for upsert operations in Delta tables
- State management: store watermark values in control table or pipeline variable
- Idempotent loads: design so re-running a load produces the same result
