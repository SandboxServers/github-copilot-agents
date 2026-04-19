## Azure Load Testing

### Overview
Fully managed service for high-scale load generation. Supports JMeter and Locust scripts. Collects Azure resource metrics during tests to identify bottlenecks across application components.

### Integration with CI/CD
- Trigger from **Azure Pipelines** or **GitHub Actions**
- Define **test fail criteria** — response time thresholds, error rate limits
- **Auto-stop** — automatically halt tests when error conditions are met
- Use in staging environments to catch performance regressions before production

### Test Design
1. Define **measurable threshold values** — requests/second, response time, error rate
2. Build **health model** based on key system flows
3. Generate **realistic traffic patterns** that represent production usage
4. Run regularly — not just before releases, but as part of continuous validation
5. Combine with **Azure Chaos Studio** for resilience testing under load

### Test Criteria
- Set **pass/fail criteria** in test configuration
- Response time (P50, P90, P99) must stay below thresholds
- Error rate must stay below threshold (typically <1%)
- Throughput must meet minimum requirements
