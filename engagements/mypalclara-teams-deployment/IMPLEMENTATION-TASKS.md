# Implementation Tasks: MyPalClara Teams Deployment

**Engagement**: mypalclara-teams-deployment
**Date**: 2026-04-20
**Status**: Ready for Implementation
**Prerequisite**: ARCHITECTURE-PLAN.md approved

---

## Phase 1: Foundation (Day 1)

| # | Task | Owner | Deliverable | Depends On | Status |
|---|---|---|---|---|---|
| 1.1 | Create resource group `mypalclara` in Central US | Platform | `az group create` output | None | Not Started |
| 1.2 | Create Log Analytics Workspace | Platform | Workspace resource ID | 1.1 | Not Started |
| 1.3 | Create Container Apps Environment | Platform | Environment resource ID | 1.1, 1.2 | Not Started |
| 1.4 | Create Azure Container Registry (Basic) | Platform | `mypalclara.azurecr.io` | 1.1 | Not Started |
| 1.5 | Create Storage Account + Azure Files shares | Platform | `qdrant-data`, `falkordb-data` shares | 1.1 | Not Started |
| 1.6 | Create Key Vault (Standard) | Security | Key Vault URI | 1.1 | Not Started |
| 1.7 | Create Entra ID App Registration (multi-tenant) | Security | App ID + Client Secret | None | Not Started |
| 1.8 | Store all secrets in Key Vault | Security | 8+ secrets stored | 1.6, 1.7 | Not Started |

## Phase 2: Data Services (Day 1-2)

| # | Task | Owner | Deliverable | Depends On | Status |
|---|---|---|---|---|---|
| 2.1 | Create PostgreSQL Flexible Server (B1ms, v16, 32GB) | Platform | Connection string | 1.1, 1.6 | Not Started |
| 2.2 | Enable pgvector extension on PostgreSQL | Platform | `CREATE EXTENSION vector` | 2.1 | Not Started |
| 2.3 | Run Alembic migrations against PostgreSQL | DevOps | Schema created | 2.1 | Not Started |
| 2.4 | Create Azure Cache for Redis (Basic C0) | Platform | Connection string | 1.1, 1.6 | Not Started |
| 2.5 | Deploy Qdrant Container App (internal, Azure Files volume) | DevOps | Qdrant responding on :6333 | 1.3, 1.5 | Not Started |
| 2.6 | Deploy FalkorDB Container App (internal, Azure Files volume) | DevOps | FalkorDB responding on :6379 | 1.3, 1.5 | Not Started |
| 2.7 | Store database connection strings in Key Vault | Security | Secrets updated | 2.1, 2.4 | Not Started |

## Phase 3: Application Deployment (Day 2-3)

| # | Task | Owner | Deliverable | Depends On | Status |
|---|---|---|---|---|---|
| 3.1 | Build and push clara-base image to ACR | DevOps | `clara-base:latest` in ACR | 1.4 | Not Started |
| 3.2 | Build and push clara-gateway image to ACR | DevOps | `clara-gateway:latest` in ACR | 3.1 | Not Started |
| 3.3 | Build and push clara-teams image to ACR | DevOps | `clara-teams:latest` in ACR | 3.1 | Not Started |
| 3.4 | Build and push clara-backup image to ACR | DevOps | `clara-backup:latest` in ACR | 3.1 | Not Started |
| 3.5 | Build and push clara-identity image to ACR | DevOps | `clara-identity:latest` in ACR | 3.1 | Not Started |
| 3.6 | Configure managed identity on all Container Apps | Security | Identity + Key Vault role assigned | 1.3, 1.6 | Not Started |
| 3.7 | Deploy Gateway Container App | DevOps | Gateway healthy, WS on :18789 | 3.2, 3.6, 2.x | Not Started |
| 3.8 | Deploy Teams Adapter Container App (external ingress) | DevOps | HTTPS endpoint available | 3.3, 3.6, 3.7 | Not Started |
| 3.9 | Deploy Backup Container App | DevOps | Backup service healthy | 3.4, 3.6 | Not Started |
| 3.10 | Deploy Identity Container App | DevOps | Identity service healthy | 3.5, 3.6 | Not Started |
| 3.11 | Validate all services healthy | DevOps | Health check pass for all 6 apps | 3.7-3.10 | Not Started |

## Phase 4: Teams Integration (Day 3)

| # | Task | Owner | Deliverable | Depends On | Status |
|---|---|---|---|---|---|
| 4.1 | Create Azure Bot Service resource (F0) | Platform | Bot resource ID | 1.7 | Not Started |
| 4.2 | Configure Bot messaging endpoint | DevOps | Endpoint pointing to Teams Adapter FQDN | 3.8, 4.1 | Not Started |
| 4.3 | Enable Microsoft Teams channel on Bot | DevOps | Teams channel active | 4.1 | Not Started |
| 4.4 | Update manifest.json with App ID | DevOps | `teams_manifest/manifest.json` updated | 1.7 | Not Started |
| 4.5 | Package Teams app (zip manifest + icons) | DevOps | `clara-teams.zip` | 4.4 | Not Started |
| 4.6 | Sideload Teams app | DevOps | App visible in Teams | 4.5, 4.2 | Not Started |
| 4.7 | Test: @mention Clara in Teams channel | All | Clara responds within 15 seconds | 4.6 | Not Started |

## Phase 5: Security & Documentation (Day 3-4)

| # | Task | Owner | Deliverable | Depends On | Status |
|---|---|---|---|---|---|
| 5.1 | Enable diagnostic settings on all resources | Security | Logs flowing to Log Analytics | 1.2, all resources | Not Started |
| 5.2 | Verify TLS 1.2+ on all connections | Security | TLS audit pass | All resources | Not Started |
| 5.3 | Document security architecture | Security | Security doc in engagement | All phases | Not Started |
| 5.4 | Write deployment runbook | DevOps | Runbook in engagement | All phases | Not Started |
| 5.5 | Establish cost baseline | Platform | First month cost projection | All resources | Not Started |
| 5.6 | Write handoff documentation | Documentation | Complete handoff doc | All phases | Not Started |

---

## Critical Path

```
1.1 → 1.3 → 2.5/2.6 (data containers)
1.1 → 2.1 → 2.2 → 2.3 (database ready)
1.4 → 3.1 → 3.2/3.3 (images built)
1.7 → 1.8 (identity ready)
All Phase 2+3 → 3.8 (Teams Adapter deployed)
3.8 → 4.2 → 4.6 → 4.7 (Teams integration tested)
```

**Estimated Duration**: 3-4 days for full implementation.
**Minimum Viable POC**: Phases 1-4 (skip Backup, Identity, Web UI) = 2 days.