## Multi-Domain Solution Design

A multi-domain solution touches network, compute, storage, database, integration, and monitoring. Designing it requires a process that prevents specialists from working in isolation and discovering incompatibilities at deployment time.

### Step 1: Understand Requirements

Before assigning specialists, understand the problem:
- What does the system do? What are the user-facing capabilities?
- What are the non-functional requirements? Availability target, latency budget, throughput, data residency, compliance.
- What are the constraints? Budget, timeline, team skills, existing infrastructure.
- What integrations exist? On-premises systems, third-party APIs, other Azure workloads.

### Step 2: Identify Azure Services

Map requirements to Azure service categories — not specific services yet:
- Data needs → Database and/or Storage domain
- Compute needs → Compute domain (request/response, event-driven, batch, GPU)
- Communication needs → Integration domain (sync, async, streaming)
- Connectivity needs → Network domain (hybrid, multi-region, isolation)
- Intelligence needs → AI domain (models, inference, RAG)

### Step 3: Map Dependencies Between Services

Draw the dependency graph. Every arrow is a constraint:
- Compute depends on networking (VNet integration, private endpoints)
- Compute depends on database (connection method, latency requirements)
- Database depends on networking (private endpoints, DNS resolution)
- Integration depends on compute (where producers and consumers run)
- AI depends on compute (GPU availability), storage (training data), and networking (endpoint access)
- Monitoring depends on everything (diagnostic settings, instrumentation)

### Step 4: Identify Cross-Cutting Concerns

Before specialists begin design work, establish the cross-cutting constraints:
- **Identity model:** Managed identity strategy, RBAC approach, Key Vault usage
- **Networking model:** VNet topology, private endpoint strategy, DNS approach
- **Monitoring model:** Log Analytics workspace, Application Insights instances, alert strategy
- **Security baseline:** Encryption requirements, WAF, Defender for Cloud
- **Naming and tagging:** Convention, enforcement policy

These are non-negotiable constraints that every specialist works within.

### Step 5: Assign Specialists

Each requirement area gets a primary specialist. Cross-cutting requirements get multiple specialists with a designated lead:
- No requirement should be orphaned — every gap is a future production issue
- Each specialist proposes specific Azure services for their domain
- The Specialty Lead reviews proposals for compatibility between domains

### Step 6: Review Integrated Design

Walk through a request from client to storage and back. At every hop:
- How does the request get there? (Networking)
- Who is the request? (Identity)
- What happens if this hop fails? (Reliability)
- Can we see it happening? (Monitoring)
- How much does this hop cost? (Cost)

If any hop can't answer all five questions, the design has a gap.

### Step 7: Validate with Security and Cost

Before implementation:
- **Security review:** Does the design meet the security baseline? Any public endpoints that shouldn't be? Any secrets not in Key Vault? Any identity gaps?
- **Cost estimate:** What's the monthly run rate? What scales with load and what's fixed? Where are the cost risks (egress, premium SKUs, reserved vs pay-as-you-go)?

### Architecture Decision Records

For every significant technology choice, document:
- **Context:** What problem are we solving?
- **Decision:** What did we choose?
- **Alternatives considered:** What else did we evaluate?
- **Consequences:** What trade-offs does this create?
- **Status:** Proposed → Accepted → Superseded

ADRs prevent revisiting decisions without new information. When someone asks "why Cosmos DB instead of PostgreSQL?" the ADR answers it.
