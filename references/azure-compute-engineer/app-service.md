## App Service Deep Dive

### Tier Comparison

| Tier | Use Case | Autoscale | Slots | VNet Integration | Custom Domain/SSL | Max Instances |
|------|----------|-----------|-------|-----------------|-------------------|---------------|
| **Free/Shared** | Dev/test only | No | No | No | Shared (no custom SSL) | 1 |
| **Basic** | Low-traffic prod | No (manual scale) | No | No | Yes | 3 |
| **Standard** | Production | Yes (rule-based) | 5 | Yes | Yes | 10 |
| **Premium v3** | High-perf production | Yes + automatic | 20 | Yes | Yes | 30 |
| **Isolated v2** | Compliance, dedicated | Yes | 20 | ASE (dedicated VNet) | Yes | 100 |

### Deployment Slot Gotchas
1. **Slots share the App Service Plan** → slot traffic competes with production for CPU/memory
2. **Swap is NOT instant** — it warms up the target slot by hitting it first, then swaps DNS
3. **App settings marked "slot setting"** stick to the slot (e.g., connection strings per environment)
4. **Swap doesn't swap**: scale settings, WebJobs schedulers, custom domain names, TLS/SSL certificates
5. **Auto-swap** can be configured for CD pipelines (push to staging → auto-swap to production)
6. **Traffic routing** — split traffic between slots for canary/A-B testing (percentage-based)

### App Service Automatic Scaling vs Autoscale
| Feature | Automatic Scaling | Autoscale (rule-based) |
|---------|------------------|----------------------|
| Tier | Premium v2+ | Standard+ |
| Trigger | HTTP traffic only | CPU, memory, queue, schedule, custom metrics |
| Configuration | Per-app | Per App Service Plan |
| Always-ready instances | Yes (minimum 1) | No |
| Prewarmed instances | Yes (buffer for cold starts) | No |
| Per-app max | Yes | No |
