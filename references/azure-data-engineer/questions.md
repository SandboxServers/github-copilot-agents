# Questions

## Discovery
- What data sources exist today? (Databases, files, APIs, SaaS apps, streaming)
- What's the data volume? (GB vs TB vs PB — this changes everything)
- What are the latency requirements? (Batch daily? Near-real-time? True streaming?)
- Who consumes the data? (BI analysts? Data scientists? Applications? External partners?)
- What governance/compliance requirements exist? (GDPR, HIPAA, data residency, retention)

## Architecture
- Is there an existing data platform? (Brownfield migration vs greenfield build)
- What skills does the team have? (SQL-heavy? Python/Spark? Power Query? All of the above?)
- What's the BI tool? (Power BI? Tableau? Both? — affects gold layer design)
- What's the budget? (This determines Fabric capacity tier, Databricks cluster size, everything)
- How many source systems? (5 sources is different architecture than 500 sources)

## Probing for Gaps
- How do you handle schema changes from source systems? (If "they break our pipeline," we need schema evolution)
- Who owns data quality? (If "nobody," that's the first problem to solve)
- How do you know if yesterday's pipeline failed? (If "we don't until someone complains," we need monitoring)
- Can you trace a number in a report back to its source? (If not, we need lineage)
- What happens when a source system is unavailable? (If "the pipeline fails," we need idempotency and retry)
