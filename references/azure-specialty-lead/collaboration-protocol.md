## When One Specialist vs a Collaboration

### Single Specialist Sufficient
- "Which Cosmos DB consistency level for our use case?" → Database Specialist alone
- "What App Service plan SKU for our workload?" → Compute Engineer alone
- "How do we configure Azure Monitor alerts for this resource?" → Monitoring Engineer alone
- "What's the best storage tier for infrequently accessed logs?" → Storage Engineer alone

### Collaboration Required
- "Design an e-commerce platform on Azure" → Compute + Database + Integration + Monitoring + Network
- "Migrate our on-prem data warehouse to Azure" → Data Engineer + Storage + Database + Network + Compute
- "Build a RAG-based AI application" → AI Specialist + Database (vector search) + Compute + Storage + Integration
- "Our API is slow under load" → Integration + Compute + Database + Monitoring (diagnosis first)
- "Design a multi-region disaster recovery strategy" → Network + Compute + Database + Storage + Monitoring

### The Collaboration Protocol
When multiple specialists are needed:
1. **Define scope**: What problem are we solving? What are the constraints?
2. **Identify the lead domain**: Which specialty owns the primary decision? Others contribute.
3. **Sequence the work**: Follow the dependency chain above
4. **Review holistically**: After each specialist contributes, review the whole — are they optimizing locally at the expense of globally?
5. **Check cross-cutting concerns**: Is networking consistent? Is identity model unified? Is monitoring comprehensive?
