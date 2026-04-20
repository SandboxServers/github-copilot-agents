# Azure Integration Architect — Knowledge Index

> **Last Updated:** 2026-04-19 — Added gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Integration Overview | [integration-overview.md](integration-overview.md) | When understanding the Azure integration stack or choosing between services |
| Logic Apps | [logic-apps.md](logic-apps.md) | When working with Logic Apps — Standard vs Consumption, error handling, transforms |
| Service Bus | [service-bus.md](service-bus.md) | When working with Service Bus — tiers, messaging patterns, DLQ, sessions |
| Event Grid | [event-grid.md](event-grid.md) | When working with Event Grid — events vs commands, saga patterns, subscriptions |
| API Management | [api-management.md](api-management.md) | When working with APIM — tiers, policies, products, developer portal |
| Hybrid Integration | [hybrid-integration.md](hybrid-integration.md) | When designing on-prem to cloud integration or BizTalk migration |
| Failure Design | [failure-design.md](failure-design.md) | When implementing retry, circuit breaker, compensation, or idempotency patterns |
| Cost Model | [cost-model.md](cost-model.md) | When estimating integration service costs or optimizing spend |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing integration designs for common mistakes |
| Gotchas | [gotchas.md](gotchas.md) | When debugging surprising behaviors or hitting hidden limits in integration services |
| Best Practices | [best-practices.md](best-practices.md) | When designing integration solutions — idempotency, sagas, retries, monitoring |
| API-First Design | [api-first-design.md](api-first-design.md) | When designing APIs contract-first, setting up API governance, or using APIM as design platform |

## Related Skills

| Skill | When to Load |
|-------|-------------|
| `api-design-standards` | Contract-first workflow, REST conventions, versioning, pagination, error responses |
| `error-handling-patterns` | Retry, circuit breaker, saga, dead-letter, idempotency patterns |
