# Real-Time Analytics Patterns

## Eventhouse and Real-Time Intelligence

### When to Use Eventhouse
Eventhouse is the preferred engine in Fabric for real-time event processing and time-series analytics. Use it for:
- **Telemetry and log data** — IoT device streams, application logs, infrastructure metrics, security audit logs.
- **Time-series analysis** — sensor readings, financial tick data, clickstream events, operational metrics.
- **Security and compliance** — threat detection, anomaly investigation, compliance audit trails.
- **Semi-structured and free text** — Eventhouse auto-indexes structured, semi-structured, and unstructured data partitioned by ingestion time.
- **High-velocity ingestion** — millions of events per second with automatic indexing and partitioning.

### Eventhouse Architecture
An Eventhouse is a workspace of KQL databases. Multiple databases share capacity and resources within a single Eventhouse, optimizing cost and performance.
- **Ingestion:** Eventstream, SDKs, Kafka, Logstash, data flows, Cosmos DB CDC via Synapse Link. Supports both real-time and batch ingestion.
- **Storage:** Automatic indexing and partitioning based on ingestion time. Data can be published to OneLake for consumption by other Fabric experiences (adds latency but enables broader integration).
- **Query:** KQL is the primary language — expressive, easy to read, optimized for time-series. T-SQL subset also supported for SQL-familiar users.
- **Tooling:** KQL Querysets for reusable, shareable queries. Copilot for natural-language-to-KQL translation. Export results to Power BI.
- **Visualization:** Real-Time Dashboards for live monitoring — distinct from Power BI. Optimized for high-velocity streaming data with drill-down, filters, and no-code exploration.

### Medallion Architecture in Real-Time Intelligence
The medallion pattern applies to streaming data via KQL database features:
- **Bronze → Silver:** Use KQL **update policies** to transform data as it arrives. Update policies handle incremental processing, checkpointing, and watermarks automatically — no extra tooling or orchestration.
- **Silver → Gold:** Use **materialized views** for real-time aggregation, deduplication, and enrichment. Always provides up-to-date results. Querying a materialized view is faster than running the aggregation over the source table.
- **Key benefit:** Update policies and materialized views eliminate the need for external orchestration tools for streaming ETL. No Spark, no ADF — just KQL declarations.
- **Cost advantage:** Materialized views consume fewer resources than repeatedly running aggregate queries.

## Eventstream — Streaming Ingestion and Routing

### Core Capabilities
Eventstream is Fabric's no-code streaming ingestion and routing engine:
- **Sources:** Azure Event Hubs, Kafka, IoT Hub, REST APIs, Fabric workspace events, OneLake events, job events, Cosmos DB CDC.
- **Processing:** Real-time filtering, data cleansing, transformation, windowed aggregations, deduplication, schema alignment, timestamp normalization.
- **Routing:** Content-based routing sends data to different destinations based on filters. Derived eventstreams create new shareable streams from transformations.
- **Destinations:** Eventhouse (KQL database), Lakehouse, Activator, custom endpoints.

### Eventstream Design Patterns
- Use Eventstream for schema alignment and timestamp normalization before landing in Eventhouse.
- Integrate with **Activator** for real-time alerting: define rules on streaming events to trigger emails, Teams messages, or Power Automate flows.
- Pause/stop/restart event flows during development without losing event position — iterate on transformation logic safely.
- Use Managed Private Endpoints for secure, private access to data sources behind firewalls or blocked from public internet.
- **Anti-pattern:** Don't use Eventstream for complex multi-step transformations — that's Spark or KQL update policy territory.

## Data Activator — Real-Time Actions and Alerts

### Alert and Action Patterns
Activator monitors streaming data and triggers actions when conditions are met:
- **From Eventstream:** Direct integration — monitor event properties against thresholds in near real-time.
- **From Eventhouse:** Subscribe to KQL queryset outputs. When a query detects a condition (e.g., CPU > 90%, error rate spike, trend indicating failure), Activator fires.
- **From Real-Time Dashboards:** No-code alert creation directly from dashboard visuals. Select a metric, set a condition, choose an action.
- **Condition types:** Simple thresholds, value trending down over time, date/time checks, basic math functions, multivariate anomaly detection.

### Action Types
- Email and Teams notifications to relevant personnel.
- Power Automate flows for complex orchestration (ticketing, escalation, remediation).
- Fabric pipeline execution for automated data remediation or re-processing.
- Low-latency triggers available for scenarios requiring sub-minute response times.

## Data Mesh Integration in Fabric

### Domain-Oriented Architecture
Fabric supports data mesh principles through its domain and workspace model:
- **Domains:** Create Fabric domains mapped to business areas (marketing, sales, finance, operations). Each domain has an owner responsible for data quality, governance, and publishing.
- **Data products:** Gold-layer tables published as discoverable, well-documented datasets via OneLake Catalog. Products include descriptions, tags, quality metrics, and freshness SLAs.
- **Federated governance:** Central platform team manages capacity allocation, security policies, Purview integration, and cross-domain standards. Domain teams own their data pipelines, transformations, and quality independently.
- **Cross-domain access:** Use OneLake shortcuts for zero-copy access across workspaces and domains. Consumers access data without ETL duplication. Storage billed to the source capacity; compute billed to the consumer capacity.

### Data Mesh + Medallion Pattern
Each domain implements its own medallion layers. The gold layer is the domain's data product interface:
- Domain teams control bronze-to-gold processing independently with their own cadence and tooling choices.
- Central governance team enforces cataloging requirements, sensitivity labels, retention policies, and naming conventions via Purview.
- OneLake Catalog provides a single discovery pane across all domains — every data product is searchable with metadata, lineage, and quality indicators.
- **Anti-pattern:** Don't create data silos by duplicating operational data across domains. Use shortcuts for cross-domain references instead of ETL copies.

## Key Decisions

### When to Use Eventhouse vs Lakehouse for Streaming
| Criteria | Eventhouse | Lakehouse |
|----------|-----------|-----------|
| Latency requirement | Sub-second to seconds | Minutes to hours |
| Query language | KQL (optimized for time-series) | Spark SQL, PySpark |
| Data shape | Events, logs, telemetry, semi-structured | Structured business data, large transforms |
| Streaming ETL | Update policies (built-in, no orchestration) | Structured Streaming (requires Spark) |
| Aggregation | Materialized views (auto-maintained) | Manual pipeline scheduling |
| Dashboard | Real-Time Dashboards (live) | Power BI (scheduled refresh) |

Choose Eventhouse when you need sub-second query latency on streaming data with KQL. Choose Lakehouse + Spark when transformations are complex and batch/micro-batch latency is acceptable.
