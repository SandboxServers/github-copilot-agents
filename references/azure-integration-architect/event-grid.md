## Event Grid

### Core Concepts
- **Event-driven, push-push model**: publishers emit events → Event Grid routes to subscribers → subscribers are pushed notifications
- Contrast with Service Bus pull model: Event Grid pushes, Service Bus is pulled (or proxied-push via Event Grid integration)
- **Topics**: system topics (Azure resource events, automatic), custom topics (your events), domain topics (grouped custom topics)
- **Event subscriptions**: route events from topic to handler with optional filters
- **Handlers**: Azure Functions, Logic Apps, webhooks, Service Bus queues/topics, Event Hubs, Storage queues

### Events vs Commands
| Aspect | Events | Commands |
|---|---|---|
| **Intent** | "Something happened" | "Do something" |
| **Coupling** | Loose — publisher doesn't know/care about subscribers | Tighter — sender expects receiver to act |
| **Delivery** | Broadcast to all subscribers | Delivered to one consumer |
| **Failure handling** | Subscriber failure doesn't affect publisher | Sender typically needs acknowledgment |
| **Example** | "Order placed" | "Process payment" |
| **Azure service** | Event Grid | Service Bus |

**Design rule**: use events for notification/reaction, commands for directed work. Many integration flows combine both — Event Grid triggers a Function that sends a command to Service Bus.

### Event Grid + Service Bus Integration
- Service Bus can emit events to Event Grid when messages arrive in a queue/subscription with no active receiver
- Enables reactive processing: instead of polling, start a receiver only when messages exist
- **Proxied push model**: Event Grid notifies → receiver starts → pulls messages from Service Bus
- Reduces cost for low-volume queues (no idle polling)

### Retry & Dead-Lettering
- Default retry: exponential backoff, 24 hours, up to 30 attempts
- Configurable: `maxDeliveryAttempts` and `eventTimeToLiveInMinutes`
- Dead-letter destination: Storage blob container for undeliverable events
- **Idempotency**: Event Grid provides at-least-once delivery — consumers MUST handle duplicates

### Saga Pattern
- Coordinate distributed transactions across services without distributed locks
- **Orchestration**: central coordinator (Logic App, Durable Functions) directs each step
- **Choreography**: each service emits events, next service reacts — no central coordinator
- **Compensation**: when a step fails, previous steps execute compensating actions (e.g., refund payment if shipping fails)
- Event Grid + Service Bus naturally support choreography; Logic Apps/Durable Functions support orchestration
