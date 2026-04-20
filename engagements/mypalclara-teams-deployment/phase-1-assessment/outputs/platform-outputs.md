# Platform Engineering Assessment: MyPalClara Teams Deployment

**Engagement**: mypalclara-teams-deployment
**Assessor**: Platform Engineering Lead
**Date**: 2026-04-20
**Phase**: Infrastructure Foundations Assessment

---

## Executive Summary

Deploy on Azure Container Apps with managed PostgreSQL/Redis and containerized vector/graph databases. Estimated cost: $45-65/month within VS Enterprise credit limits.

## Infrastructure Service Mapping

| Application Component | Recommended Azure Service | Rationale |
|---|---|---|
| **Gateway** (Python/FastAPI) | Container Apps | Serverless scaling, cost-effective, built-in ingress |
| **Teams Adapter** (Bot Framework) | Container Apps | HTTPS endpoint requirement, auto-scaling |
| **Backup Service** | Container Apps | Scheduled execution for periodic tasks |
| **Identity Service** | Container Apps | Internal service, minimal compute needs |
| **PostgreSQL 16** | Database for PostgreSQL Flexible Server | Managed, SOC2 compliant, automated backup |
| **Redis 7** | Azure Cache for Redis | Managed, SOC2 compliant |
| **Qdrant** (Vector DB) | Container Apps | No managed Azure equivalent, containerized |
| **FalkorDB** (Graph DB) | Container Apps | No managed Azure equivalent, containerized |

## Compute Platform: Azure Container Apps

**Rationale**:
- Serverless billing with scale-to-zero for POC workload
- Built-in HTTPS ingress with TLS termination (required for Bot Framework)
- Native container-to-container networking within environment
- Managed identity support for SOC2 compliance
- Minimal infrastructure overhead vs AKS

**Rejected alternatives**:
- **AKS**: ~$73/month baseline cluster cost exceeds POC budget
- **App Service**: Limited multi-container orchestration, higher per-app cost

## Database Strategy

### Managed Services

| Service | SKU | Cost/month | Notes |
|---|---|---|---|
| PostgreSQL Flexible Server | Burstable B1ms (1 vCPU, 2GB) | ~$16 | pgvector extension available, automatic backups |
| Azure Cache for Redis | Basic C0 (250MB) | ~$16 | Sufficient for embedding cache |

### Containerized (No Managed Equivalent)

| Service | Deployment | Storage | Notes |
|---|---|---|---|
| Qdrant | Container App | Azure Files persistent volume | 1024-dim BGE-large embeddings |
| FalkorDB | Container App | Azure Files persistent volume | Graph memory store |

## Networking

```
Internet → Container Apps Ingress (TLS) → Teams Adapter (:3978/api/messages)
                                            ↓ WebSocket
                          Internal VNet → Gateway (:18789 WS, :18790 HTTP)
                                            ↓
                          Private/Internal → PostgreSQL, Redis, Qdrant, FalkorDB
```

- Container Apps Environment provides built-in internal networking
- Teams Adapter gets external HTTPS ingress (Bot Framework messaging endpoint)
- Gateway and databases are internal-only
- PostgreSQL Flexible Server: VNet integration or private endpoint
- Redis Cache: Private endpoint where cost-effective

## Identity & RBAC

| Component | Identity Method |
|---|---|
| Container Apps → Key Vault | System-assigned managed identity |
| Container Apps → PostgreSQL | Connection string via Key Vault |
| Container Apps → Redis | Connection string via Key Vault |
| Bot Framework → Teams | Entra ID app registration (multi-tenant) |
| Teams Adapter → Gateway | Internal networking + shared secret |

### Entra ID App Registration
- Create multi-tenant app registration for Bot Framework
- Store client secret in Key Vault
- Bot Service resource references the app registration

## Region Recommendation: Central US

- All required services available: Container Apps, PostgreSQL Flexible, Redis Cache, Bot Service, Key Vault, Container Registry
- Good quota availability for Container Apps
- Central location reduces latency to Teams infrastructure
- **Fallback**: West US 3 or South Central US if quota issues

## Cost Estimate (Monthly)

| Service | SKU/Tier | Est. Cost |
|---|---|---|
| Container Apps (6 apps) | Consumption plan, ~0.25 vCPU avg | $8-15 |
| PostgreSQL Flexible Server | Burstable B1ms + 32GB | $16 |
| Azure Cache for Redis | Basic C0 | $16 |
| Azure Files (persistent vols) | Standard, 64GB | $5 |
| Key Vault | Standard | $1-3 |
| Container Registry | Basic | $5 |
| Azure Bot Service | F0 (Free) | $0 |
| **Total** | | **$51-60/month** |

Within $150/month VS Enterprise budget with ~$90 headroom.

## Risks

| Risk | Severity | Mitigation |
|---|---|---|
| Qdrant/FalkorDB container persistence | High | Validate Azure Files mount + backup before data load |
| Teams Adapter cold start | Medium | Set min replicas = 1 to avoid Bot Framework timeout |
| PostgreSQL pgvector extension | Low | Available on Flexible Server, verify on Burstable tier |
| Regional quota limits | Low | Central US has good defaults; fallback regions identified |

## Dependencies

- **Security**: Entra ID app registration review, Key Vault access patterns, SOC2 controls
- **DevOps**: Container image build strategy, deployment automation
- **Cost Optimization**: Validate sizing assumptions, commitment discount opportunities

---

**Status**: Assessment complete. Ready for architecture synthesis.

---