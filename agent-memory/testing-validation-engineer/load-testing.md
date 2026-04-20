## Load Testing

### Azure Load Testing Service

- Fully managed, high-scale load generation service
- Based on Apache JMeter — use existing JMeter scripts or create new ones
- Collects server-side metrics from Azure resources during test execution
- Supports distributed load generation across multiple test engines
- Integrates with Application Insights for correlated diagnostics

### CI/CD Integration

- Trigger load tests from Azure Pipelines or GitHub Actions
- Define pass/fail criteria in test configuration — pipeline fails if thresholds are breached
- Run automatically on staging deployments before production promotion
- Auto-stop capability — halt tests when error conditions are met early
- Store test results as pipeline artifacts for historical comparison

### Test Criteria

| Metric | Typical Threshold | Notes |
|--------|------------------|-------|
| **Response time P50** | <200ms | Median user experience |
| **Response time P95** | <1s | Most users' worst case |
| **Response time P99** | <3s | Tail latency for edge cases |
| **Error rate** | <1% | Non-2xx responses or timeouts |
| **Throughput** | Meets SLA target | Requests per second at expected concurrency |

### Baseline Establishment

- Run load tests against a known-good version to establish baseline metrics
- Store baseline results and compare every subsequent run against them
- Flag performance regressions automatically — P95 response time increased by >10%
- Baselines should be refreshed after significant architecture changes
- Track baselines per environment (staging baseline differs from production)

### Types of Performance Testing

| Type | Goal | Duration | Load Pattern |
|------|------|----------|-------------|
| **Load test** | Validate system under expected load | 10-30 min | Steady at target concurrency |
| **Stress test** | Find breaking point and failure behavior | 15-60 min | Ramp up beyond capacity |
| **Soak test** | Detect memory leaks and resource exhaustion | 2-24 hours | Steady at moderate load |
| **Spike test** | Validate auto-scaling and burst handling | 5-15 min | Sudden traffic spike |

### Identifying Bottlenecks

Common bottleneck sources during load testing:
- **CPU saturation** — compute-bound processing, inefficient algorithms
- **Memory pressure** — memory leaks, large object heap growth, GC pauses
- **Connection pool exhaustion** — database connections, HTTP client connections
- **Throttling** — Azure resource limits (DTUs, RUs, IOPS, API rate limits)
- **Network latency** — cross-region calls, DNS resolution, TLS handshake overhead
- **Lock contention** — database locks, distributed lock conflicts

### Test Data Management

- Use synthetic data that matches production data distribution and volume
- Avoid using production data directly — mask or anonymize sensitive fields
- Pre-populate test databases before load test runs
- Clean up test data after runs to keep environments consistent
- Parameterize test scripts to avoid cache-friendly patterns that skew results

### Environment Isolation

- Run load tests in dedicated environments — never test against production
- Isolate load test environments from other testing to avoid interference
- Match production configuration as closely as possible (SKUs, scaling rules, networking)
- Monitor resource consumption during tests to right-size test environments
- Use separate Application Insights instances to avoid polluting production telemetry
