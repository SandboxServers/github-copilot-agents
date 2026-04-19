## Azure Technology Choice Decision Points

### Compute Decision Tree
- **Request/response web workloads**: App Service → Container Apps → AKS (escalating complexity)
- **Event-driven / serverless**: Functions (Flex Consumption) → Container Apps (KEDA) → AKS
- **Batch / HPC**: Azure Batch → AKS with spot pools → VMs
- **GPU / AI workloads**: AKS with GPU pools → Azure ML compute → VMs with GPU
- **Lift and shift**: VMs → VMware Solution → App Service migration

### Data Decision Tree
- **Relational**: SQL Database → PostgreSQL Flexible Server → MySQL Flexible Server
- **Document / NoSQL**: Cosmos DB (multiple APIs) → MongoDB on VM
- **Key-value / Cache**: Azure Managed Redis → Cosmos DB Table API
- **Graph**: Cosmos DB Gremlin API → SQL Server graph features
- **Time series**: Azure Data Explorer → Cosmos DB with TTL
- **Data warehouse**: Microsoft Fabric → Synapse Analytics → Databricks

### Integration Decision Tree
- **Simple async**: Storage Queue → Service Bus Queue
- **Pub/Sub**: Service Bus Topics → Event Grid → Event Hubs
- **Event streaming at scale**: Event Hubs → Kafka on HDInsight
- **API management**: API Management → Azure Front Door (simple routing)
- **Workflow orchestration**: Logic Apps → Durable Functions → AKS

### Storage Decision Tree
- **Object storage**: Blob Storage (Hot/Cool/Cold/Archive)
- **File shares**: Azure Files (SMB/NFS) → Azure NetApp Files (high performance)
- **Data lake**: ADLS Gen2 (hierarchical namespace on Blob)
- **Disk storage**: Managed Disks (Premium SSD v2 → Premium SSD → Standard SSD)
