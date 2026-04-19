## End-to-End Solution Design Framework

When designing a multi-domain solution, the Specialty Lead follows this framework:

### 1. Requirements Decomposition
Break the solution into domain-specific requirements:
- What data needs to be stored, how much, how fast, how long?
- What compute is needed — request/response, batch, event-driven, GPU?
- What integration patterns — sync APIs, async messaging, event streaming?
- What network constraints — hybrid, multi-region, compliance boundaries?
- What AI/ML workloads — training, inferencing, RAG, agents?

### 2. Domain Assignment
Map requirements to specialists:
- Each requirement gets a primary owner (specialist)
- Cross-cutting requirements get multiple specialists with a designated lead
- No requirement should be orphaned — every gap is a future production issue

### 3. Dependency Mapping
Identify what depends on what:
- Draw the dependency graph — networking at the bottom, workloads at the top
- Identify circular dependencies (these indicate design problems)
- Sequence work to minimize rework

### 4. Technology Selection
Each specialist proposes services for their domain. The Specialty Lead evaluates:
- Do the selected services work together? (VNet integration compatibility, identity model alignment)
- Are there region constraints? (Some SKUs or services not available in all regions)
- Is the cost model understood? (Egress, cross-region, premium SKU costs)
- Is the operational model sustainable? (Team has skills, tooling, and bandwidth to operate this)

### 5. Holistic Review
Before implementation:
- Walk through a request from client to storage and back — does every hop work?
- Check all five Well-Architected pillars
- Verify monitoring covers every component
- Confirm identity model is consistent
- Review cost model end-to-end (not just per-service estimates)
- Validate disaster recovery covers the full solution, not just individual services
