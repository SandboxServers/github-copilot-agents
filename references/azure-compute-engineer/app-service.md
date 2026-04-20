# App Service Deep Dive

## Tier Comparison

| Tier | Use Case | Autoscale | Deployment Slots | VNet Integration | Custom Domain/SSL | Max Instances | Approx Min Cost |
|------|----------|-----------|-----------------|-----------------|-------------------|---------------|-----------------|
| **Free (F1)** | Dev/test only | No | No | No | No custom SSL | 1 | $0 |
| **Shared (D1)** | Dev/test only | No | No | No | Yes (custom domain) | 1 | ~$10/mo |
| **Basic (B1-B3)** | Low-traffic production | No (manual only) | No | No | Yes | 3 | ~$55/mo |
| **Standard (S1-S3)** | Production workloads | Yes (rule-based) | 5 slots | Yes (regional) | Yes + managed certs | 10 | ~$73/mo |
| **Premium v3 (P0v3-P3mv3)** | High-performance production | Yes + Automatic | 20 slots | Yes (regional + private endpoints) | Yes + managed certs | 30 | ~$78/mo (P0v3) |
| **Isolated v2 (I1v2-I6v2)** | Compliance, dedicated network | Yes | 20 slots | Yes (ASE, full VNet isolation) | Yes + managed certs | 100 | ~$298/mo |

## App Service Plan Architecture

- **App Service Plan** = the compute unit (VMs running your apps)
- **Multiple apps share one plan** — apps on the same plan compete for CPU, memory, disk
- Plan determines: region, VM size, instance count, OS (Linux/Windows)
- **Scale unit**: all apps in a plan scale together (same instance count)
- Pack multiple low-traffic apps per plan to maximize value
- Separate high-traffic or resource-intensive apps to their own plans

## Linux vs Windows

| Factor | Linux | Windows |
|--------|-------|---------|
| **.NET 8+** | Recommended (better performance) | Supported |
| **.NET Framework (4.x)** | Not supported | Required |
| **Node.js** | Recommended | Supported |
| **Python** | Recommended | Supported (limited) |
| **Java** | Recommended | Supported |
| **PHP** | Recommended | Supported |
| **Ruby** | Supported | Not supported |
| **Custom containers** | Supported | Supported (Premium+) |
| **SSH access** | Yes (built-in) | No |
| **Cost** | Same (but Linux has P0v3 entry tier) | Same |

**General guidance:** Use Linux unless you need .NET Framework or Windows-specific dependencies.

## Deployment Slots

Available on Standard tier and above. Key behaviors:

- Slots share the App Service Plan — slot traffic competes with production for resources
- **Swap is not instant** — Azure warms the target slot first, then swaps the VIP
- **Slot settings** (marked "deployment slot setting") stick to the slot, not the app. Use for environment-specific connection strings and app settings
- **Swap does NOT swap:** scale settings, WebJobs schedules, custom domains, TLS certificates, diagnostic settings
- **Auto-swap**: push to staging → automatic swap to production (useful for CD pipelines)
- **Traffic routing**: split traffic between slots by percentage for canary/A-B testing

## Scaling Options

| Method | Available Tier | How It Works | Best For |
|--------|---------------|-------------|----------|
| **Manual scale** | Basic+ | Set fixed instance count | Predictable, steady load |
| **Autoscale (rule-based)** | Standard+ | Rules on CPU, memory, HTTP queue, schedule, custom metrics | Variable load with known patterns |
| **Automatic scaling** | Premium v3+ | Azure manages instances based on HTTP traffic | Unpredictable HTTP traffic |

**Automatic scaling features:**
- Per-app scaling (independent of plan)
- Always-ready instances (minimum warm instances)
- Pre-warmed instances (buffer for cold starts)
- Maximum burst limit per app

## Health Checks & Auto-Heal

**Health check path**: Configure a URL that App Service pings every minute. Unhealthy instances (failing > 5 checks) are replaced. Set `WEBSITE_HEALTHCHECK_MAXPINGFAILURES` to tune threshold.

**Auto-heal rules** (Custom rules to recycle app on conditions):
- Slow requests (> X seconds for Y requests)
- HTTP status code frequency (e.g., 500 errors > 100 in 5 minutes)
- Memory limit exceeded
- Actions: recycle process, log event, or custom action

## Deployment Methods

| Method | Use Case | Zero-Downtime |
|--------|----------|--------------|
| **ZIP deploy** | CI/CD pipelines (GitHub Actions, Azure DevOps) | With deployment slots |
| **Git deploy (local git)** | Small teams, simple repos | No (builds on server) |
| **Container deploy** | Custom runtime, Docker images | With deployment slots |
| **GitHub Actions / Azure DevOps** | Automated CI/CD | With slot swap step |
| **Run from package** | Immutable deployments, fastest start | With deployment slots |

**Best practice:** Deploy to staging slot → validate → swap to production = zero-downtime deployment.

## Networking

- **Regional VNet integration** (Standard+): Outbound traffic routes through VNet. Access private resources (databases, storage with private endpoints)
- **Private endpoints** (Premium v3+): Inbound traffic restricted to VNet. No public IP exposure
- **Access restrictions**: IP-based or service-tag-based allow/deny rules on the public endpoint
- **Hybrid Connections**: Access on-premises resources without VPN (relay-based, no VNet needed)

## Common Mistakes

- One app per Premium plan running at 5% CPU — pack multiple apps or downsize the plan
- Using Isolated v2 for "VNet access" — Premium v3 with regional VNet integration + private endpoints is cheaper
- Not using deployment slots for production deployments — direct deploy causes downtime during restart
- Free/Shared tier for production — shared infrastructure, no SLA, no custom SSL
- Ignoring health check configuration — unhealthy instances serve traffic until manual intervention
- Not setting `WEBSITES_ENABLE_APP_SERVICE_STORAGE` to false for container apps — unnecessary disk sync
