## Integration Anti-Patterns

1. **Synchronous everything** — don't call 5 backends synchronously in sequence when they could be parallelized or async
2. **No dead-letter monitoring** — dead letters accumulate silently, indicating broken integration
3. **Retry without idempotency** — retries cause duplicate processing, data corruption
4. **One giant Logic App** — split workflows by bounded context, not by "everything in one place"
5. **Ignoring message ordering** — competing consumers break ordering. Use sessions if order matters.
6. **APIM as passthrough only** — if you're not using policies, you're paying for a proxy. Add value: auth, rate limiting, caching, transformation.
7. **Polling when events exist** — use Event Grid reactive model instead of polling Service Bus/storage for changes
8. **No compensation logic** — distributed transactions need rollback plans. Design compensation from day one.
