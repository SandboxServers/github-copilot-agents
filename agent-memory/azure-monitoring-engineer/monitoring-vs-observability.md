# Monitoring vs Observability

Monitoring and observability are related but distinct concepts. Understanding the difference guides how you instrument and operate systems.

## Monitoring

- Predefined dashboards and alerts for known failure modes
- Answers known questions: "Is CPU above 90%?" "Is the service up?"
- Deals with **known-unknowns** — things you expect might go wrong
- Reactive: alert fires → investigate → fix
- Based on predefined thresholds and expected patterns

## Observability

- Ability to ask new questions about system behavior without deploying new code
- Answers unknown questions: "Why is checkout slow for users in Europe?"
- Deals with **unknown-unknowns** — things you didn't predict
- Proactive: explore data → discover patterns → prevent issues
- Based on rich, high-cardinality, high-dimensionality data

## The Journey: Monitoring → Observability

1. **Start with metrics + alerts** — basic monitoring. Know when things break.
2. **Add structured logging** — context for investigation. Know why things break.
3. **Add distributed tracing** — flow analysis. Know where things break in a chain.
4. **Add custom events and dimensions** — business context. Know what impact breakage has.
5. **Enable ad-hoc queries** — full observability. Ask any question about system behavior.

## Implementation in Azure

| Capability | Azure Service | Purpose |
|-----------|--------------|----------|
| Metrics for known patterns | Azure Monitor Metrics | Dashboards, alerting, autoscaling |
| Logs for investigation | Log Analytics (KQL) | Root cause analysis, compliance |
| Traces for flow analysis | Application Insights | Cross-service request tracking |
| Custom events/dimensions | Application Insights SDK | Business-specific questions |
| Interactive analysis | Workbooks | Parameterized exploration |
| Ad-hoc queries | KQL in Log Analytics | Ask arbitrary questions |

## Key Insight

Monitoring tells you when something is broken. Observability helps you understand why — even for problems you've never seen before. A mature platform needs both.
