# Gotchas

Surprising behaviors and hidden limits in Azure integration services. These catch experienced engineers.

## Service Bus: Message Lock Duration vs Processing Time

- Default lock duration is 60 seconds — if processing takes longer, the lock expires and another consumer picks up the same message
- Renewing the lock (`RenewMessageLockAsync`) must happen before expiry — if your code blocks on a long operation, the renewal never fires
- **Trap:** lock renewal timers don't survive process crashes — the message redelivers after lock expiry regardless
- Maximum lock duration is 5 minutes — for longer processing, use sessions or defer the message and process asynchronously
- `MaxAutoLockRenewalDuration` in the SDK controls automatic renewal — set it to your worst-case processing time + margin
- Dead-letter after `MaxDeliveryCount` (default 10) — if locks keep expiring, you'll hit this silently

## Service Bus: Dead Letter Queue Messages Don't Expire

- Messages moved to the DLQ have no TTL by default — they accumulate indefinitely
- DLQ counts against the queue's storage quota (1–80 GB depending on tier)
- **Trap:** a silently filling DLQ can cause the main queue to reject new messages when storage is full
- Standard tier: 80 GB per queue. Premium tier: 100 GB per messaging unit
- Build automated DLQ processors — read, log, alert, and either resubmit or archive
- Monitor `deadLetterMessageCount` metric — alert on any value > 0

## Service Bus: Sessions Lock Entire Session to One Consumer

- When sessions are enabled, a session is locked to a single consumer for the lock duration
- Other consumers cannot process messages for that session — even if they're idle
- **Trap:** hot sessions (one session ID with many messages) create bottlenecks while other consumers starve
- Session state is limited to 256 KB — don't use it as a general-purpose store
- Premium tier supports up to 10,000 concurrent sessions; Standard supports 100

## Service Bus: Message Size Limits by Tier

- Standard tier: maximum message size 256 KB (including headers and properties)
- Premium tier: maximum message size 100 MB
- **Trap:** a message that works in Premium fails silently or throws in Standard after a tier migration
- Use the claim-check pattern for large payloads — store payload in blob, send reference in message

## Event Grid: At-Least-Once Delivery Requires Idempotent Handlers

- Event Grid guarantees at-least-once delivery — duplicates are expected, not exceptional
- The same event can be delivered multiple times during retries or infrastructure rebalancing
- **Trap:** non-idempotent handlers (e.g., incrementing a counter) produce incorrect results
- Use the `id` field on each event for deduplication — store processed IDs in a fast lookup (Redis, table)
- Event Grid retries for up to 24 hours with exponential backoff

## Event Grid: Event Ordering Is NOT Guaranteed

- Events may arrive out of order — even events from the same source
- **Trap:** assuming ordered delivery causes state machine corruption or incorrect sequencing
- Use `eventTime` or a custom sequence number to detect and handle out-of-order events
- If strict ordering is required, route through Service Bus sessions instead of Event Grid

## Event Grid: Subscription Validation Handshake

- When creating a webhook subscription, Event Grid sends a `SubscriptionValidation` event with a `validationCode`
- Your endpoint must return the code in the response to prove ownership — otherwise the subscription fails
- **Trap:** custom endpoints behind authentication reject the validation request — you must allow unauthenticated access to the validation path
- Alternative: use the `validationUrl` (manual validation) — fetch the URL within 5 minutes to confirm

## Logic Apps: HTTP Trigger Timeout Limits

- Consumption plan: HTTP request trigger times out at 4 minutes (240 seconds)
- Standard plan: HTTP request trigger times out at 2 minutes (120 seconds)
- **Trap:** Standard has a shorter timeout than Consumption — migration breaks long-running sync calls
- For long operations, return 202 Accepted with a polling URL (async pattern) instead of waiting

## Logic Apps: ForEach Concurrency Affects Variable Scoping

- When `foreach` concurrency is > 1, iterations run in parallel
- **Trap:** `Append to array variable` or `Increment variable` actions inside parallel foreach produce race conditions and non-deterministic results
- Variables in Logic Apps are workflow-scoped, not iteration-scoped — concurrent writes corrupt state
- Use `Select` action to transform arrays instead, or set concurrency to 1 for variable mutations

## Logic Apps Standard: Stateful vs Stateless Workflow Differences

- Stateful workflows persist run history and support resubmission — stateless do not
- Stateless workflows have a 5-minute execution timeout (vs no hard limit for stateful)
- **Trap:** stateless workflows don't support breakpoints in debugging, chunked transfer, or managed connector triggers
- Use stateful for orchestration and stateless for high-throughput request-response patterns

## APIM: Rate Limiting Is Approximate

- `rate-limit` and `rate-limit-by-key` policies use distributed counters across gateway instances
- **Trap:** actual throughput may exceed the configured limit by 10-20% during counter synchronization windows
- For strict enforcement, combine with backend-side throttling as a safety net
- `quota` policies are also approximate — use for governance, not hard limits

## APIM: Backend Timeout Defaults to 30 Seconds

- The `forward-request` policy defaults to 30 seconds for backend calls
- **Trap:** long-running backend operations (reports, exports) fail with 504 Gateway Timeout
- Set `timeout` attribute on `forward-request` explicitly — maximum is 240 seconds
- For operations longer than 240 seconds, implement the async request-reply pattern with polling

## APIM: Consumption Tier Cold Start

- APIM Consumption tier has no always-running infrastructure — first call after idle period takes 1-3 seconds
- **Trap:** timeout-sensitive clients fail on the cold start delay, especially health check probes
- Subsequent calls in the same period are fast — cold start only affects idle gateways
- No mitigation other than switching to a provisioned tier (Basic v2+) for latency-sensitive APIs

## Logic Apps: Connector Authentication Token Expiry

- OAuth-based connectors (Office 365, Salesforce, SharePoint) store refresh tokens that can expire or be revoked
- **Trap:** workflows silently fail weeks after deployment when the token expires — no proactive warning
- Use managed identity authentication where supported — no tokens to expire
- For OAuth connectors, monitor for `AuthorizationFailed` errors and set up alerts
- Re-authorize connectors through the Logic Apps designer — cannot be done via API or CLI

## Service Bus: Prefetch Count Tuning

- `PrefetchCount` controls how many messages the client fetches ahead of processing
- **Trap:** high prefetch with slow processing causes message lock expiry on prefetched messages — they redeliver to other consumers
- Set prefetch to 0 for long-processing workloads, or to `MaxConcurrentCalls × 20` for high-throughput scenarios
- Prefetched messages are locked immediately — they count against your lock duration budget

## Event Grid: Custom Topic Throughput Limits

- Custom topics: 5,000 events/sec or 5 MB/sec ingress (whichever is hit first)
- **Trap:** burst traffic exceeding the limit gets throttled with 429 responses — events are not queued
- System topics (Azure resource events) have higher limits managed by the platform
- For higher throughput, partition across multiple custom topics or use Event Hubs instead
