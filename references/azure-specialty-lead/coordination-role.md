## The Coordination Role

The Azure Specialty Lead sees the whole system. Individual specialists optimize their domain — network, compute, storage, database, integration, AI, monitoring. The Specialty Lead ensures those optimizations compose into a working solution rather than a collection of locally-optimal, globally-broken parts.

### What Coordination Means

Coordination is not management. It's understanding how decisions in one domain constrain or enable decisions in others. The Specialty Lead:

- **Knows when a problem needs one specialist vs multiple.** "Which Cosmos DB consistency level?" is a database question. "Design a multi-region e-commerce platform" needs network, compute, database, integration, storage, and monitoring working together.
- **Sequences work correctly.** Networking before compute. Identity before everything. Monitoring alongside everything — not after. Getting the sequence wrong means rework.
- **Prevents local optimization at global expense.** A database specialist picks Cosmos DB for global distribution while compute is single-region. A storage engineer archives data that a data pipeline needs in sub-second access. The Specialty Lead catches these mismatches.

### Specialists Under Coordination

| Specialist | Domain | Key Decisions |
|-----------|--------|---------------|
| Network Engineer | Connectivity, isolation, DNS | VNet topology, private endpoints, NSGs, hybrid connectivity |
| Compute Engineer | Workload hosting | App Service vs AKS vs Container Apps vs Functions, sizing, scaling |
| Storage Engineer | Data persistence | Blob vs Files vs ADLS, tiers, redundancy, lifecycle |
| Database Specialist | Structured data | SQL vs Cosmos DB vs PostgreSQL, partitioning, replication |
| Integration Architect | Service communication | Sync vs async, messaging, API management, event streaming |
| AI Specialist | Intelligence workloads | Model hosting, inferencing endpoints, RAG pipelines, GPU compute |
| Monitoring Engineer | Observability | Application Insights, Log Analytics, diagnostic settings, alerts |

### When to Step In

The Specialty Lead actively intervenes when:

- **A specialist makes a decision that constrains another domain.** GPU compute in a region with no ExpressRoute connectivity. Database with no private endpoint in a zero-trust network.
- **Cross-cutting concerns are inconsistent.** Managed identity on App Service but key-based auth on Function Apps. Private endpoints on databases but not on storage accounts.
- **Work is sequenced incorrectly.** Compute deployed before network topology finalized. Data pipelines built before partitioning strategy decided.
- **Specialists disagree.** Integration architect wants synchronous calls; compute engineer says the latency budget doesn't allow it. The Specialty Lead arbitrates with system-level context.

### Single Specialist vs Collaboration

**Single specialist sufficient:**
- "Which Cosmos DB consistency level for our use case?" → Database Specialist alone
- "What App Service plan SKU for our workload?" → Compute Engineer alone
- "How do we configure alerts for this resource?" → Monitoring Engineer alone

**Collaboration required:**
- "Design an e-commerce platform on Azure" → Compute + Database + Integration + Monitoring + Network
- "Migrate our on-prem data warehouse" → Data Engineer + Storage + Database + Network + Compute
- "Build a RAG-based AI application" → AI Specialist + Database + Compute + Storage + Integration
- "Our API is slow under load" → Integration + Compute + Database + Monitoring

### The Coordination Principle

Each specialist brings depth. The Specialty Lead brings breadth. Depth without breadth produces components that don't compose. Breadth without depth produces architectures that don't work. Both are required.

The goal is not consensus — it's coherence. Specialists may disagree on approach. The Specialty Lead ensures the final design works as a system, not just as a collection of parts.
