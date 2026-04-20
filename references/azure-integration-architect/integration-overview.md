# Integration Overview

## Azure Integration Services Decision Matrix

| Service | Pattern | Best For | Pricing Model |
|---|---|---|---|
| **Logic Apps** | Workflow orchestration | SaaS integration, business process automation, B2B/EDI | Per-action (Consumption) or vCPU/memory (Standard) |
| **Service Bus** | Messaging | Async messaging, decoupling, ordered delivery, transactions | Per-operation + base tier (Standard) or per-MU (Premium) |
| **Event Grid** | Eventing | Reactive event routing, pub/sub, fan-out, notifications | Per-event ($0.60/million after 100K free) |
| **API Management** | API gateway | API facade, rate limiting, auth, monetization, developer portal | Per-tier (Consumption to Premium) |
| **Functions** | Compute | Event processing, lightweight transforms, glue code | Per-execution (Consumption) or plan-based (Premium/Dedicated) |
| **Data Factory** | Data movement | ETL/ELT, batch data pipelines, data integration | Per-activity run + data movement + pipeline orchestration |

## Integration Patterns

### Messaging Patterns
- **Request-reply**: Synchronous or correlated async request with response. Service Bus sessions for correlation.
- **Pub/sub**: One publisher, many subscribers. Event Grid for notifications, Service Bus topics for reliable delivery.
- **Competing consumers**: Multiple receivers on a single queue for load balancing. Service Bus queues.
- **Dead letter**: Failed messages routed to separate queue for investigation. Every queue/topic must have DLQ monitoring.
- **Claim check**: Store large payload in blob storage, pass reference in message. Keeps messages small.
- **Content-based routing**: Route messages based on content/properties. Service Bus topic filters or Event Grid advanced filtering.

### Transaction Patterns
- **Saga (choreography)**: Each service emits events, next service reacts. No central coordinator. Event Grid + Service Bus.
- **Saga (orchestration)**: Central coordinator directs each step. Logic Apps or Durable Functions as orchestrator.
- **Compensation**: Each step has a compensating action for rollback. Design compensation from day one.

### Event-Driven vs Command-Driven
| Aspect | Events | Commands |
|---|---|---|
| **Intent** | "Something happened" (notification) | "Do something" (instruction) |
| **Coupling** | Loose — publisher doesn't know subscribers | Tighter — sender expects receiver to act |
| **Delivery** | Broadcast to all subscribers (fan-out) | Delivered to one consumer (point-to-point) |
| **Failure** | Subscriber failure doesn't affect publisher | Sender typically needs acknowledgment |
| **Guarantee** | At-least-once (idempotent consumers required) | At-least-once with PeekLock |
| **Azure service** | Event Grid | Service Bus |

**Design rule**: Use events for notification/reaction, commands for directed work. Many flows combine both — Event Grid triggers a Function that sends a command to Service Bus.

## Architecture Patterns

### Basic Enterprise Integration
```
Client → APIM → Logic Apps → Backend System
```
Synchronous, simple. Good for low-complexity request/response APIs wrapping legacy backends.

### Message Broker + Events (Recommended for Production)
```
Client → APIM → Logic Apps → Service Bus → Backend Processors
                           → Event Grid → Notification Handlers
```
Async, reliable, scalable. Decoupled producers and consumers. Dead-letter handling for failures.

### Event-Driven Microservices
```
Service A → Event Grid → Service B, C, D (reactive)
Service A → Service Bus → Service E (reliable, ordered)
```
Event Grid for reactive fan-out. Service Bus for reliable delivery with ordering guarantees.

### Hybrid Integration
```
Cloud Services → Logic Apps + On-Prem Data Gateway → On-Prem Systems
                 Service Bus → Hybrid Connections → On-Prem Services
                 APIM Self-Hosted Gateway → On-Prem APIs
```
Multiple connectivity options for on-premises systems.

## When to Combine Services

| Scenario | Combination |
|---|---|
| API with async processing | APIM → Functions → Service Bus → Backend |
| SaaS integration with notifications | Logic Apps → Service Bus + Event Grid |
| Multi-tenant event routing | Event Grid domains → Service Bus per-tenant queues |
| B2B with partner APIs | APIM + Logic Apps + Integration Account |
| Hybrid with API management | APIM (self-hosted gateway) + Service Bus + Logic Apps |

## Core Principles

1. **Prefer async over sync**: Async messaging prevents cascading failures and improves resilience
2. **Design for failure**: Every integration point can fail. Retry, dead-letter, compensate.
3. **Idempotency everywhere**: At-least-once delivery means consumers must handle duplicates
4. **Monitor everything**: Queue depth, DLQ count, message age, throughput, error rates
5. **Start simple, evolve**: Begin with Event Grid for simple events. Add Service Bus when you need ordering, transactions, or reliability guarantees.
6. **Loose coupling**: Events over direct calls. Queues over synchronous chains. Contracts over implementations.
