## Azure Functions Deep Dive

### Hosting Plan Comparison

| Plan | Scale | Cold Start | Max Instances | Execution Limit | Scale-to-Zero | Best For |
|------|-------|------------|---------------|----------------|---------------|----------|
| **Consumption** | Auto (event-driven) | Yes (seconds) | 200 | 10 min (default), 30 min max | Yes | Sporadic, low-traffic event processing |
| **Flex Consumption** | Auto (event-driven) | Reduced (always-ready) | 1,000 | Configurable | Yes (with always-ready option) | High-scale event processing, VNet |
| **Premium (EP)** | Auto (event-driven) | No (prewarmed) | 100 (Linux), 60 (Win) | Unlimited | No (min 1 instance) | Low-latency, VNet, longer execution |
| **Dedicated (App Service)** | Manual / autoscale | No | Per ASP limits | Unlimited | No | Existing ASP with spare capacity |
| **Container Apps** | KEDA-based | Depends on min replicas | 1,000 | Unlimited | Yes | Custom containers, polyglot |

### Cold Start Mitigation Strategies
1. **Premium plan** — pre-warmed instances always running (but costs money)
2. **Flex Consumption** — always-ready instances configurable per function
3. **Consumption** — use timer trigger to keep warm (hack, not recommended)
4. **Reduce package size** — smaller deployment = faster cold start
5. **Avoid heavy initialization** — lazy-load dependencies, use singleton patterns
6. **Language matters**: .NET (compiled) < Node.js < Python < Java (coldest)

### Functions Execution Limits to Know
| Limit | Consumption | Premium | Dedicated |
|-------|------------|---------|-----------|
| Max execution time | 10 min (default) | Unlimited | Unlimited |
| Max HTTP response time | 230 seconds | 230 seconds | 230 seconds |
| Max outbound connections | 600 active (1200 total) | Unbounded | Depends on plan |
| Max request size | 100 MB | 100 MB | 100 MB |
| Max instances | 200 | 100 | Per ASP |
