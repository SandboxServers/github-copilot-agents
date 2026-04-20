# Azure Functions

## Hosting Plan Comparison

| Plan | Scale Model | Cold Start | Max Instances | Execution Limit | Scale-to-Zero | VNet | Best For |
|------|------------|------------|---------------|----------------|---------------|------|----------|
| **Consumption** | Auto (event-driven) | Yes (1-10s) | 200 | 10 min default, 30 min max | Yes | Outbound only | Sporadic, low-traffic event processing |
| **Flex Consumption** | Auto (event-driven) | Reduced (always-ready option) | 1,000 | Configurable | Yes (with always-ready option) | Full VNet + private endpoints | High-scale event processing, VNet-integrated |
| **Premium (EP1-EP3)** | Auto (event-driven) | No (pre-warmed) | 100 (Linux), 60 (Windows) | Unlimited | No (min 1 instance) | Full VNet + private endpoints | Low-latency, VNet, long execution |
| **Dedicated (App Service)** | Manual / autoscale rules | No | Per ASP limits | Unlimited | No | Per ASP tier | Existing ASP with spare capacity |
| **Container Apps** | KEDA-based | Depends on min replicas | 1,000 | Unlimited | Yes | Environment VNet | Custom containers, polyglot runtimes |

## Triggers Reference

| Trigger | Source | Common Use Case |
|---------|--------|----------------|
| **HTTP** | HTTP request | APIs, webhooks, REST endpoints |
| **Timer** | CRON schedule | Scheduled jobs, cleanup tasks, report generation |
| **Blob Storage** | New/updated blob | File processing, image resize, ETL |
| **Queue Storage** | Queue message | Async task processing, decoupled workflows |
| **Service Bus** | Queue/topic message | Enterprise messaging, ordered processing |
| **Event Grid** | Event subscription | Resource events, custom events, reactive patterns |
| **Event Hubs** | Event stream | Telemetry ingestion, log processing, IoT |
| **Cosmos DB** | Change feed | Real-time materialized views, event sourcing |
| **SignalR** | SignalR negotiation | Real-time web (negotiate endpoint) |
| **Kafka** | Kafka topic | Stream processing (Confluent or Event Hubs Kafka) |
| **Dapr** | Dapr binding/pub-sub | Microservices integration |

## Durable Functions

Orchestration patterns for stateful, long-running workflows:

| Pattern | Description | Use Case |
|---------|------------|----------|
| **Function chaining** | Sequential execution: A → B → C | Multi-step processing pipelines |
| **Fan-out/fan-in** | Parallel execution, aggregate results | Parallel document processing, batch operations |
| **Async HTTP API** | Long-running operation with status polling | Operations > 230 seconds, background jobs |
| **Monitor** | Periodic polling with flexible intervals | Waiting for external conditions, approval workflows |
| **Human interaction** | Wait for external event (human approval) | Approval workflows, escalation with timeout |
| **Aggregator (entity)** | Stateful entity, accumulate events over time | Running totals, session tracking, device state |

**Entity functions:** Stateful actors that persist state across invocations. Use for counters, device state, shopping carts — anything needing durable mutable state.

## Execution Limits

| Limit | Consumption | Flex Consumption | Premium | Dedicated |
|-------|------------|-----------------|---------|-----------|
| Max execution time | 10 min (30 min configurable) | Configurable | Unlimited | Unlimited |
| Max HTTP response time | 230 seconds | 230 seconds | 230 seconds | 230 seconds |
| Max outbound connections | 600 active (1,200 total) | Higher | Unbounded | Depends on ASP |
| Max request size | 100 MB | 100 MB | 100 MB | 100 MB |
| Max instances | 200 | 1,000 | 100 (Linux) | Per ASP |
| Storage account required | Yes (Azure Files) | Yes | Yes | Yes |

## Cold Start Mitigation

| Strategy | Plan | How It Works |
|----------|------|-------------|
| **Pre-warmed instances** | Premium | Min 1 instance always running. No cold start. Costs money 24/7 |
| **Always-ready instances** | Flex Consumption | Configurable per-function trigger. Pay for always-ready instances only |
| **Timer keep-alive** | Consumption | Timer trigger every 5 min to keep instance warm. Hack — not recommended |
| **Reduce package size** | All | Smaller deployment packages = faster cold start. Trim dependencies |
| **Avoid heavy init** | All | Lazy-load dependencies. Use singleton/static patterns for HTTP clients, DB connections |
| **Choose .NET** | All | Compiled languages cold-start faster. Order: .NET < Node.js < Python < Java |

## Best Practices

- **Keep functions small and focused** — one trigger, one responsibility. Don't build monolith functions
- **Avoid long-running functions** — use Durable Functions for orchestration. Consumption plan has 10-min limit
- **Use managed identity** — never store connection strings in app settings. Use `@Microsoft.KeyVault()` references or identity-based connections
- **Connection pooling** — use static/singleton `HttpClient`, `CosmosClient`, `ServiceBusClient`. New instance per invocation = port exhaustion
- **Idempotent design** — functions may retry on failure. Design for at-least-once delivery semantics
- **Structured logging** — use `ILogger` with structured properties. Query in Application Insights with Kusto
- **Function app per domain** — group related functions in one app. Don't create one app per function (scaling unit is the app)
- **Output bindings over SDK calls** — simpler code, automatic retry, better integration
- **Monitor with Application Insights** — enable by default. Track invocations, failures, duration, dependencies

## Common Mistakes

- Functions Consumption for latency-sensitive APIs → cold starts ruin P99 latency. Use Premium or Flex
- Synchronous blob triggers for high-volume → Event Grid trigger is faster and more reliable
- One function app per function → all functions in an app scale together, but too many apps waste resources
- `HttpClient` created per invocation → socket exhaustion under load. Use static instance
- Premium plan for rarely-executed functions → paying $155+/month for occasional triggers. Use Consumption
- Not setting `host.json` concurrency limits → uncontrolled parallelism can overwhelm downstream services
- Ignoring Flex Consumption → newer plan that solves VNet + cold start + scale-to-zero together
