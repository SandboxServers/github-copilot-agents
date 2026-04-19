## Work Sequencing — The Dependency Chain

The order in which specialists work matters. Getting the sequence wrong means rework.

### Phase 1: Foundation (Network + Identity)
- **Network Engineer**: Design VNet topology, subnets, NSGs, private DNS zones, connectivity (ExpressRoute/VPN)
- **Identity**: Plan managed identity strategy, RBAC model, Key Vault architecture
- These must be in place before any other domain deploys resources

### Phase 2: Data Layer (Database + Storage)
- **Database Specialist**: Design database architecture — service selection, partitioning, replication, backup
- **Storage Engineer**: Design storage architecture — account structure, access tiers, lifecycle policies, redundancy
- Data layer depends on networking (private endpoints) and identity (RBAC, managed identity access)

### Phase 3: Compute + Integration
- **Compute Engineer**: Select and size compute platforms — App Service, AKS, Container Apps, Functions
- **Integration Architect**: Design messaging, API management, event-driven patterns
- Compute depends on networking (VNet integration) and data layer (connection strings, endpoints)
- Integration depends on compute (where code runs) and data (what it connects to)

### Phase 4: Intelligence + Workloads
- **AI Specialist**: Deploy AI/ML models, configure inferencing endpoints, connect to data pipelines
- **Data Engineer**: Build data pipelines, medallion architecture, ETL/ELT flows
- AI depends on compute (GPU availability), storage (training data), and networking (endpoint access)
- Data engineering depends on storage, compute, and integration patterns

### Phase 5: Observability (from day one, not after)
- **Monitoring Engineer**: Unified observability — Application Insights, Log Analytics, diagnostic settings across all resources
- Monitoring must be designed alongside phases 1-4, not bolted on in phase 5
- Every resource deployed in earlier phases should have diagnostic settings from creation
