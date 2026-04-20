# Architecture Plan: MyPalClara Teams Deployment

**Engagement**: mypalclara-teams-deployment
**Version**: 1.0
**Date**: 2026-04-20
**Status**: Planning Complete — Awaiting Implementation Approval
**Synthesized from**: Platform Engineering, Security & Compliance, DevOps assessments

---

## Architecture Overview

Deploy MyPalClara AI chatbot to Azure Container Apps with managed database services, integrated with Microsoft Teams via Bot Framework. Single resource group, single region, SOC2-ready controls.

```
                            ┌─────────────────────┐
                            │   Microsoft Teams    │
                            └──────────┬──────────┘
                                       │ HTTPS
                            ┌──────────▼──────────┐
                            │  Azure Bot Service   │
                            │  (F0 Free Tier)      │
                            └──────────┬──────────┘
                                       │
                    ┌──────────────────▼──────────────────┐
                    │     Container Apps Environment       │
                    │              (Central US)             │
                    │                                      │
                    │  ┌────────────┐  ┌──────────────┐   │
           HTTPS ──►│  │   Teams    │  │   Gateway    │   │
                    │  │  Adapter   │──│  (FastAPI)   │   │
                    │  │  (:3978)   │WS│ (:18789/90)  │   │
                    │  └────────────┘  └──────┬───────┘   │
                    │                         │           │
                    │  ┌─────────┐  ┌─────────▼────────┐  │
                    │  │ Backup  │  │ Identity Service │  │
                    │  │ Service │  │    (:18791)      │  │
                    │  └─────────┘  └──────────────────┘  │
                    │                                      │
                    │  ┌─────────┐  ┌──────────────────┐  │
                    │  │ Qdrant  │  │    FalkorDB      │  │
                    │  │ (Vector)│  │    (Graph)       │  │
                    │  └─────────┘  └──────────────────┘  │
                    └──────────────────┬──────────────────┘
                                       │
                    ┌──────────────────▼──────────────────┐
                    │         Managed Services             │
                    │                                      │
                    │  ┌──────────────┐ ┌──────────────┐  │
                    │  │  PostgreSQL  │ │  Redis Cache │  │
                    │  │  Flex (B1ms) │ │  (Basic C0)  │  │
                    │  └──────────────┘ └──────────────┘  │
                    │                                      │
                    │  ┌──────────────┐ ┌──────────────┐  │
                    │  │  Key Vault   │ │  Container   │  │
                    │  │  (Standard)  │ │  Registry    │  │
                    │  └──────────────┘ └──────────────┘  │
                    └─────────────────────────────────────┘
```

## Decision Record

### DR-1: Compute Platform — Azure Container Apps
**Decision**: Use Container Apps (Consumption plan) over AKS or App Service
**Rationale**:
- Scale-to-zero for POC cost savings
- Built-in HTTPS ingress with TLS termination (required for Bot Framework)
- Internal networking between containers
- Managed identity support
- ~$8-15/month vs ~$73/month AKS baseline

### DR-2: Region — Central US
**Decision**: Deploy all resources in Central US
**Rationale**:
- All required services available (Container Apps, PostgreSQL Flex, Redis, Bot Service, Key Vault, ACR)
- Good default quota availability
- Fallback: West US 3 or South Central US

### DR-3: Database Strategy — Managed + Containerized
**Decision**: Managed PostgreSQL + Redis; containerized Qdrant + FalkorDB
**Rationale**:
- PostgreSQL and Redis have managed Azure equivalents (Flexible Server, Cache for Redis)
- Qdrant and FalkorDB have no managed Azure equivalent — deploy as Container Apps with Azure Files volumes
- Managed services provide automated backups and SOC2 compliance

### DR-4: Secrets — All in Key Vault
**Decision**: All sensitive configuration in Azure Key Vault
**Rationale**: Security requirement (H-1, H-2 findings). No secrets in environment variables or config files.
**Pattern**: Container Apps → managed identity → Key Vault secret references

### DR-5: Teams Integration — Existing Adapter + Azure Bot Service
**Decision**: Use existing `mypalclara/adapters/teams/` code with Azure Bot Service resource
**Rationale**:
- Full Teams adapter already implemented (botbuilder-core 4.16)
- Teams manifest already exists (v1.16 schema)
- Only infrastructure needed: Azure Bot Resource + Entra ID app registration
- RSC permissions (no tenant-wide admin consent)

### DR-6: No Web UI for POC
**Decision**: Skip Rails + React web UI deployment
**Rationale**: Teams bot is the target interface. Web UI adds significant complexity (Ruby, Node.js) for no POC value.

---

## Resource Inventory

| Resource | Azure Service | SKU/Tier | Est. Monthly Cost |
|---|---|---|---|
| Container Apps Environment | Container Apps | Consumption | Included |
| Teams Adapter | Container App | 0.25 vCPU, 0.5Gi, min 1 | ~$3 |
| Gateway | Container App | 0.5 vCPU, 1Gi, min 1 | ~$5 |
| Qdrant | Container App | 0.25 vCPU, 0.5Gi, min 1 | ~$3 |
| FalkorDB | Container App | 0.25 vCPU, 0.5Gi, min 1 | ~$3 |
| Backup | Container App | 0.25 vCPU, 0.5Gi, min 0 | ~$1 |
| Identity | Container App | 0.25 vCPU, 0.5Gi, min 0 | ~$1 |
| PostgreSQL Flexible Server | Database for PostgreSQL | Burstable B1ms, 32GB | ~$16 |
| Azure Cache for Redis | Redis Cache | Basic C0 | ~$16 |
| Azure Container Registry | ACR | Basic | ~$5 |
| Key Vault | Key Vault | Standard | ~$1-3 |
| Azure Files | Storage Account | Standard LRS, 64GB | ~$5 |
| Azure Bot Service | Bot Service | F0 (Free) | $0 |
| Entra ID App Registration | Microsoft Entra ID | Included | $0 |
| Log Analytics Workspace | Monitor | Pay-per-GB | ~$1-3 |
| **TOTAL** | | | **$60-64/month** |

**Budget**: $150/month VS Enterprise credits. Headroom: ~$86-90/month.

---

## Security Controls

### Mandatory (Before Deployment)

| Control | Implementation |
|---|---|
| Secrets in Key Vault | All API keys, connection strings, bot credentials |
| Managed identity | System-assigned on all Container Apps → Key Vault access |
| TLS 1.2+ | Container Apps ingress, PostgreSQL, Redis |
| RBAC scoping | Key Vault Secrets User role (not Administrator) |
| Audit logging | Diagnostic settings → Log Analytics, 12+ month retention |

### Post-Deployment (30 days)

| Control | Implementation |
|---|---|
| Data classification | Chat = Confidential, Embeddings = Internal |
| Network segmentation | Document service communication flows |
| Data purging | User data deletion capability |

---

## Networking

### External Endpoints

| Endpoint | Purpose | Authentication |
|---|---|---|
| `https://teams-adapter.<env>.centralus.azurecontainerapps.io/api/messages` | Bot Framework messaging | Bot Framework token validation |

### Internal Communication

| Source | Destination | Protocol | Port |
|---|---|---|---|
| Teams Adapter | Gateway | WebSocket | 18789 |
| Gateway | PostgreSQL Flexible Server | TCP/TLS | 5432 |
| Gateway | Redis Cache | TCP/TLS | 6380 |
| Gateway | Qdrant | HTTP | 6333 |
| Gateway | FalkorDB | TCP | 6379 |
| Gateway | External LLM APIs | HTTPS | 443 |

---

## Identity Architecture

```
Entra ID App Registration (multi-tenant)
├── Used by: Azure Bot Service + Teams Adapter
├── Client ID → Key Vault → TEAMS_APP_ID
└── Client Secret → Key Vault → TEAMS_APP_PASSWORD

Container App: Teams Adapter
├── System-assigned managed identity
├── Key Vault Secrets User role
└── Pulls secrets at startup

Container App: Gateway
├── System-assigned managed identity
├── Key Vault Secrets User role
└── Pulls database connection strings + API keys
```

---

## Deployment Sequence

```
Phase 1: Foundation
├── Resource Group
├── Log Analytics Workspace
├── Key Vault + secrets
├── Container Apps Environment
├── Container Registry
├── Storage Account + file shares
└── Entra ID App Registration

Phase 2: Data Services
├── PostgreSQL Flexible Server (+ pgvector extension)
├── Azure Cache for Redis
├── Qdrant Container App (+ Azure Files volume)
└── FalkorDB Container App (+ Azure Files volume)

Phase 3: Application
├── Build + push container images to ACR
├── Deploy Gateway Container App
├── Deploy Teams Adapter Container App
├── Deploy Backup + Identity Container Apps
└── Validate internal connectivity

Phase 4: Teams Integration
├── Azure Bot Service resource
├── Configure messaging endpoint
├── Enable Teams channel
├── Package + sideload Teams manifest
└── Test: bot responds to @mention
```

---

## Risk Register

| Risk | Probability | Impact | Mitigation | Owner |
|---|---|---|---|---|
| Qdrant/FalkorDB data loss (no managed backup) | Medium | High | Azure Files snapshots, application-level backup | DevOps |
| Teams Adapter cold start > 15s | Medium | High | min replicas = 1, validate response times | DevOps |
| Regional quota exhaustion | Low | Medium | Fallback regions identified (West US 3, South Central US) | Platform |
| LLM API costs exceed budget | Medium | Medium | Monitor usage, set spending alerts | Cost |
| pgvector extension unavailable on B1ms | Low | High | Verify during provisioning, upgrade tier if needed | Platform |

---

## Success Validation

| Criterion | Test | Pass Condition |
|---|---|---|
| App running on Azure | All Container Apps healthy | `provisioningState: Succeeded` for all 6 apps |
| Teams manifest registered | Manifest sideloaded | App appears in Teams app catalog |
| Bot endpoint configured | POST to `/api/messages` | HTTP 200 response |
| Bot responds | @mention in Teams channel | Clara replies within 15 seconds |

---