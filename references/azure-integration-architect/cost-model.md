# Integration Cost Model

## Service Pricing Summary

| Service | Pricing Model | Dev/Test Cost | Production Cost |
|---|---|---|---|
| **Logic Apps Consumption** | Per action (~$0.000125/action) + per trigger | Very low at low volume | Can be expensive at high volume with loops |
| **Logic Apps Standard** | Fixed WS1/WS2/WS3 plan | Predictable monthly | Better value for consistent/high volume |
| **Service Bus Standard** | Per operation (~$0.05/M) + base charge | Low | Good for moderate throughput |
| **Service Bus Premium** | Per messaging unit (MU) | ~$668/month (1 MU) | Scale MUs for throughput (~4 MB/s per MU) |
| **Event Grid** | Per event ($0.60/M after 100K free) | Nearly free | Very cheap at scale |
| **APIM Consumption** | Per 10K calls (first 1M free/month) | Nearly free | Cost grows with volume |
| **APIM Developer** | Per unit/hour | ~$49/month | No SLA — dev/test only |
| **APIM Standard** | Per unit/hour | — | ~$326/month |
| **APIM Standard v2** | Per unit/hour + overage | — | Better price-performance than classic |
| **APIM Premium** | Per unit/hour | — | ~$2,794/month per unit |
| **Functions Consumption** | Per execution + GB-s | Nearly free (1M free) | Cheap for moderate volume |
| **Functions Premium** | Per vCPU/GB + pre-warmed instances | ~$147/month minimum | Predictable, no cold starts |

## Cost Traps

### Logic Apps Consumption Loops
- Each action in each iteration counts as a billable execution.
- A for-each loop over 1,000 items with 5 actions = 5,000 action executions per run.
- With retries, multiply again. A daily workflow can become surprisingly expensive.
- **Fix**: Switch to Logic Apps Standard for consistent high-volume workflows.

### APIM Tier Over-Provisioning
- Premium when Standard v2 suffices. Premium is ~$2,794/month per unit.
- Multiple Premium units for availability when one unit with availability zones works.
- **Fix**: Start with Standard v2. Only use Premium for multi-region, full VNet injection, or self-hosted gateway.

### Service Bus Premium When Standard Works
- Premium required ONLY for: VNet/private endpoints, large messages (>256 KB), availability zones, geo-DR, predictable latency.
- If you don't need these features, Standard is significantly cheaper (per-operation vs ~$668/month fixed).
- **Fix**: Use Standard unless you have a specific Premium-only requirement.

### Unused or Idle Resources
- APIM instances running with no traffic (especially classic tiers — still billed hourly).
- Logic Apps Standard plans with minimal workflow usage.
- Service Bus Premium namespaces with low throughput.
- **Fix**: Use Consumption tiers for variable/low workloads. Shut down dev/test instances when not in use.

## Cost Optimization Strategies

### Right-Size Service Selection

| Scenario | Cost-Effective Choice |
|---|---|
| Simple event fan-out | Event Grid (cheapest per-event) |
| Low-volume API gateway | APIM Consumption (pay-per-call) |
| Simple transforms/glue code | Functions Consumption (nearly free) |
| Occasional workflows | Logic Apps Consumption (pay-per-action) |
| High-volume messaging | Service Bus Standard (per-operation) |
| High-volume workflows | Logic Apps Standard (fixed plan) |
| Enterprise API platform | APIM Standard v2 (good price-performance) |

### Logic Apps Optimization
- **Batch operations**: Use batch triggers and array processing instead of per-item loops.
- **Stateless workflows** (Standard): Faster, cheaper for request-response patterns. No run history overhead.
- **Built-in connectors** over managed: Run in-process, no per-action managed connector charge.
- **Terminate early**: Use conditions to skip unnecessary processing.

### APIM Optimization
- **Caching**: Cache backend responses to reduce backend calls and improve latency.
- **Consumption tier**: For APIs with variable, low-to-moderate traffic.
- **Response compression**: Reduce bandwidth costs.

### Service Bus Optimization
- **Batch send/receive**: Reduce per-operation costs by batching messages.
- **Premium auto-scale**: Scale down during low-traffic periods. Don't over-provision MUs.
- **Message size**: Keep messages small. Use claim-check pattern for large payloads.

### General Optimization
- **Event Grid over Service Bus** for simple notification/fan-out (much cheaper per event).
- **Functions over Logic Apps** for simple transforms and glue logic (per-execution cheaper than per-action).
- **Monitor and right-size monthly**: Azure Cost Management alerts for integration service spend.
