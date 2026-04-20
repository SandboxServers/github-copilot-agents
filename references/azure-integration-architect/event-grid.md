# Event Grid

## Core Concepts

Event Grid is a push-push eventing service. Publishers emit events → Event Grid routes to subscribers → subscribers are pushed notifications. Contrast with Service Bus pull model.

## Event Sources

### System Topics (Azure Services)
- Automatic for Azure services. Subscribe to resource events without custom code.
- **Storage**: BlobCreated, BlobDeleted — trigger processing on file upload
- **Resource Manager**: ResourceWriteSuccess, ResourceDeleteSuccess — audit and automation
- **IoT Hub**: DeviceCreated, DeviceTelemetry — IoT event processing
- **Container Registry**: ImagePushed, ImageDeleted — CI/CD triggers
- **Key Vault**: SecretNewVersionCreated, CertificateNearExpiry — secret rotation automation
- **App Configuration**: KeyValueModified — configuration change propagation

### Custom Topics
- Your own events. Publish via HTTP POST to topic endpoint.
- Authentication: Access keys or Azure AD (RBAC) for publishing.
- Use for domain events in microservice architectures.

### Event Domains
- Group thousands of custom topics under one domain. Per-domain access control.
- Multi-tenant event routing: one topic per tenant within a domain.
- Single endpoint for publishing, automatic topic creation per event type.

### Partner Events
- Third-party SaaS events (Auth0, SAP, etc.) delivered via Event Grid.
- Subscribe to partner events the same way as Azure system events.

## Event Handlers

| Handler | Use Case |
|---|---|
| **Azure Functions** | Serverless event processing, transforms |
| **Logic Apps** | Workflow orchestration triggered by events |
| **Webhooks** | Custom HTTP endpoints, external systems |
| **Service Bus** | Reliable delivery, ordering via queue/topic |
| **Event Hubs** | High-throughput telemetry and event streaming |
| **Storage Queues** | Simple queue-based processing |

## Event Schema

- **CloudEvents v1.0** (preferred): Industry standard. Portable across cloud providers. Recommended for new implementations.
- **Event Grid schema** (legacy): Azure-specific. Still supported, works fine for Azure-only scenarios.
- Both schemas include: event type, source, ID, time, data payload.

## Filtering

- **Event type filtering**: Subscribe only to specific event types (e.g., BlobCreated, not BlobDeleted).
- **Subject filtering**: Prefix and/or suffix matching on event subject (e.g., `/blobServices/default/containers/images/`).
- **Advanced filtering**: Filter on data fields with operators (StringContains, NumberGreaterThan, IsNotNull, etc.). Up to 25 advanced filters per subscription.
- Filtering reduces handler invocations and cost. Filter as aggressively as possible.

## Retry and Dead-Letter

### Retry Policy
- Default: Exponential backoff, max 30 attempts over 24 hours.
- Configurable per subscription: `maxDeliveryAttempts` (1-30) and `eventTimeToLiveInMinutes` (1-1440).
- Events dropped after max attempts or TTL expiration.

### Dead-Letter
- Configure dead-letter destination: Storage blob container.
- Undeliverable events written as blobs with full event data and delivery failure details.
- **ALWAYS configure dead-letter** for production subscriptions. Without it, failed events are silently dropped.
- Monitor dead-letter container for failed deliveries.

### Idempotency
- Event Grid provides **at-least-once delivery**. Consumers MUST handle duplicates.
- Use event ID for deduplication if needed. Design handlers to be naturally idempotent.

## Event Grid + Service Bus Integration

- Service Bus emits events to Event Grid when messages arrive with no active receiver.
- **Proxied push model**: Event Grid notifies → receiver starts → pulls messages from Service Bus.
- Reduces cost for low-volume queues (no idle polling). Enables reactive consumption.

## Monitoring

- **Delivery metrics**: Success/failure counts per subscription. Published event counts per topic.
- **Dead-letter storage**: Inspect undelivered events in blob container.
- **Diagnostic logs**: Publish and delivery failures with error details.
- **Alerts**: Configure on delivery failure rate, dead-letter events, publish failures.
