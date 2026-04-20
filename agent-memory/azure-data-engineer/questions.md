# Probing Questions

Questions to ask when designing a data platform.

## Sizing and Requirements

- "What's the data volume and expected growth rate?" → Determines storage tier and compute sizing
- "Real-time or batch? What latency is acceptable?" → Streaming vs batch ingestion architecture
- "Who consumes the data? BI users, data scientists, applications?" → Shapes gold layer design
- "What compliance requirements exist? (GDPR, HIPAA, data residency)" → Purview, encryption, regional deployment
- "What's the budget model: consumption-based or reserved capacity?" → Fabric capacity vs Databricks DBU pricing

## Platform Selection

- "Existing investment in Spark or Databricks?" → Databricks continuity vs Fabric migration
- "Is there a multi-cloud requirement?" → Databricks (multi-cloud) over Fabric (Azure-only)
- "What's the BI tool? Power BI, Tableau, both?" → Fabric has native Power BI; Databricks needs connectors
- "What skills does the team have? SQL, Python/Spark, Power Query?" → Platform and tooling alignment
- "How many data sources and what connector types?" → Data Factory evaluation for complex integrations

## Architecture and Quality

- "Is this greenfield or migrating from an existing platform?" → Migration complexity assessment
- "Data quality requirements and current quality ownership?" → Quality rules at silver layer, tooling choice
- "How do you handle schema changes from source systems?" → Schema evolution strategy needed
- "Retention requirements for raw data and processed data?" → Lifecycle management, Delta VACUUM policies
- "Can you trace a number in a report back to its source?" → Lineage requirements (Purview/Unity Catalog)

## Operational Readiness

- "How do you know if yesterday's pipeline failed?" → Monitoring and alerting strategy
- "What happens when a source system is unavailable?" → Idempotency, retry, and resilience design
- "Who owns data quality today?" → If nobody, governance is the first problem to solve
- "How many environments needed? (dev, test, staging, prod)" → Capacity planning and CI/CD design
- "What's the recovery time objective if the platform goes down?" → DR strategy and backup requirements
