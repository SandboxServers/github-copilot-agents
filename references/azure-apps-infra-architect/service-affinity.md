# Service Affinity Patterns

Reference stacks — services that work well together.

### Web Application Stack

App Service + SQL Database + Key Vault + Application Insights + Front Door + Redis Cache

Why: Managed PaaS across the board. App Service integrates natively with Key Vault references. App Insights auto-instruments .NET/Java/Node. Front Door provides global load balancing + WAF + CDN.

### Microservices Stack

Container Apps + Cosmos DB + Service Bus + API Management + Application Insights

Why: Container Apps handles service scaling independently. Cosmos DB for globally distributed data. Service Bus for async messaging. APIM for API gateway with rate limiting and auth.

### Enterprise Kubernetes Stack

AKS + Container Registry + Key Vault + Azure Monitor + Azure Policy + Defender for Containers

Why: Full control with enterprise guardrails. ACR for image hosting with geo-replication. Key Vault CSI driver for secrets. Azure Policy for cluster governance.

### Serverless Event Processing

Functions + Event Grid + Storage Queues + Logic Apps + Cosmos DB

Why: Event Grid for reactive events. Functions for processing. Storage Queues or Service Bus for durable messaging. Logic Apps for SaaS integrations.

### Data Platform Stack

Fabric/Synapse + ADLS Gen2 + Purview + Data Factory + Power BI

Why: Lakehouse architecture. ADLS Gen2 for raw storage. Purview for governance. Data Factory for orchestration when Fabric notebooks aren't enough.

### AI Application Stack

Azure OpenAI + AI Search + Cosmos DB + App Service + Application Insights

Why: OpenAI for inference. AI Search for RAG vector store. Cosmos DB for conversation history. App Service for the application layer.
