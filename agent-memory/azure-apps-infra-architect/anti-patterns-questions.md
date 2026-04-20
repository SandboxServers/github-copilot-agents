# Anti-Patterns & Probing Questions

## Architecture Anti-Patterns
| Anti-Pattern | Problem | Better Approach |
|---|---|---|
| Monolith on AKS | Over-engineered, expensive, complex | App Service or Container Apps |
| Microservices for a CRUD app | Unnecessary complexity | Monolith on App Service |
| VMs for a web API | Unmanaged OS, patching burden | App Service or Container Apps |
| Single region for production | Single point of failure | Multi-region with Front Door |
| Secrets in app settings | Exposed in portal, logs, deployments | Key Vault references |
| Public endpoints for internal APIs | Unnecessary attack surface | Private endpoints + VNet integration |
| No health endpoints | Blind to app state | /health and /ready endpoints |
| Premium everything in dev | Wasted spend | Smallest viable SKU, auto-shutdown |
| Manual deployments | Inconsistent, error-prone, audit-less | CI/CD with IaC |
| Shared databases across services | Tight coupling, deployment bottleneck | Database per service |

## Probing Questions
Ask these when designing or reviewing an architecture:

### Scale & Performance
- "What's the expected request rate at peak?" → Sizing decisions
- "What's the acceptable response time (p95/p99)?" → Caching, CDN, async
- "Do you need to scale to zero?" → Container Apps or Functions vs App Service

### Compliance & Security
- "What compliance frameworks apply?" (SOC2, HIPAA, PCI) → Isolated environments, encryption, audit
- "Does data need to stay in a specific region?" → Data residency constraints
- "Are there network isolation requirements?" → Private endpoints, ASE, VNet integration

### Team & Operations
- "Does the team have Kubernetes expertise?" → AKS vs Container Apps
- "What's the deployment frequency?" → CI/CD maturity
- "What's the budget?" → SKU selection, commitment discounts
- "Who will operate this at 2 AM?" → Managed services preferred

### Data
- "What's the data model?" → SQL vs NoSQL vs both
- "How much data and how fast does it grow?" → Storage tiering, retention
- "Multi-region writes needed?" → Cosmos DB
