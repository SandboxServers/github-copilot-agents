## Chaos Engineering

### Principles of Chaos Engineering

Chaos engineering is the discipline of experimenting on a system to build confidence in its ability to withstand turbulent conditions in production.

1. **Define steady state** — what does "normal" look like? (response time, error rate, throughput)
2. **Hypothesize** — "the system will continue operating normally when X fails"
3. **Introduce failure** — inject a real-world fault (network partition, pod kill, disk pressure)
4. **Observe** — measure deviation from steady state
5. **Learn and improve** — fix weaknesses discovered, update runbooks, add monitoring

### Azure Chaos Studio

- Managed service for running chaos experiments against Azure resources
- **Agent-based faults** — inject failures inside VMs (CPU pressure, memory stress, process kill, network latency)
- **Service-direct faults** — disrupt Azure services externally (Cosmos DB failover, AKS pod chaos, NSG rules)
- Target resources using selectors — resource IDs, tags, or resource groups
- Integrates with Azure Monitor for observing experiment impact
- RBAC controlled — experiments require explicit permissions on target resources

### Fault Injection Experiments

| Fault Type | What It Tests | Example |
|-----------|---------------|---------|
| **Pod/process kill** | Service restart and recovery | Kill a single pod, verify replacement spins up |
| **Network latency** | Timeout handling and retries | Add 500ms latency to database calls |
| **Network partition** | Failover and circuit breakers | Block traffic between services |
| **DNS failure** | DNS fallback and caching | Disrupt DNS resolution for a dependency |
| **CPU/memory stress** | Auto-scaling and degradation | Consume 90% CPU on a node |
| **AZ/region failure** | Zone/region redundancy | Simulate availability zone outage |
| **Dependency unavailability** | Graceful degradation | Block access to a downstream API |

### Steady State Hypothesis

- Define measurable indicators of normal system behavior before each experiment
- Examples: error rate <1%, P95 latency <500ms, all health checks passing
- Monitor these indicators during and after the experiment
- The experiment succeeds if the system maintains steady state despite the fault

### Blast Radius Control

- **Start small** — always limit the scope of the first experiment
- Target a single instance, pod, or AZ before expanding
- Use RBAC and experiment permissions to prevent accidental broad impact
- Set abort conditions — automatically stop if impact exceeds expectations
- Run experiments in non-production environments first, then graduate to production

### Progressive Experiment Growth

1. **Single pod kill** — does the service recover? How long?
2. **Multiple pod kills** — does the replica set maintain availability?
3. **Node failure** — does the scheduler redistribute workloads?
4. **AZ failure** — does cross-zone redundancy work?
5. **Region failover** — does DR activation meet RTO/RPO targets?

### Gameday Exercises

- Scheduled team exercises that simulate production incidents
- Combine chaos experiments with human response procedures
- Test monitoring, alerting, escalation paths, and communication channels
- Rotate the facilitator — everyone should experience being on-call during chaos
- Document findings: what broke, what was detected, what was missed, action items

### Document and Improve

- Every chaos experiment produces a report: hypothesis, observations, findings
- Track improvements over time — same experiment should show better results
- Update runbooks and playbooks based on what was learned
- Add monitoring for blind spots discovered during experiments
- Share findings across teams — resilience is an organizational capability
