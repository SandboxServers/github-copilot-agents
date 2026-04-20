# DevOps Assessment: MyPalClara Teams Deployment

**Engagement**: mypalclara-teams-deployment
**Assessor**: DevOps Lead
**Date**: 2026-04-20
**Phase**: Container Build & Deployment Strategy

---

## Executive Summary

Deploy via shared base image strategy with Azure Container Registry. Manual deployment sufficient for POC with operational runbooks.

## Container Build Strategy

### Build Architecture

```
clara-base (services/base/Dockerfile)
├── Python 3.12-slim + Poetry + all deps
└── Shared by all services

clara-gateway (services/gateway/Dockerfile) — FROM clara-base
clara-teams (same gateway image, different entrypoint)
clara-backup (services/backup/Dockerfile) — FROM clara-base
clara-identity (services/identity/Dockerfile) — FROM clara-base
```

### Azure Container Registry

- Registry: `mypalclara.azurecr.io` (Basic tier, ~$5/month)
- Single region, no geo-replication needed
- Admin user disabled — use managed identity for Container Apps pull

### Build Process

```bash
# 1. Build base image
docker build -f services/base/Dockerfile -t mypalclara.azurecr.io/clara-base:latest .
docker push mypalclara.azurecr.io/clara-base:latest

# 2. Build service images
docker build -f services/gateway/Dockerfile -t mypalclara.azurecr.io/clara-gateway:latest .
docker tag mypalclara.azurecr.io/clara-gateway:latest mypalclara.azurecr.io/clara-teams:latest

docker build -f services/backup/Dockerfile -t mypalclara.azurecr.io/clara-backup:latest .
docker build -f services/identity/Dockerfile -t mypalclara.azurecr.io/clara-identity:latest .

# 3. Push all
for svc in gateway teams backup identity; do
  docker push mypalclara.azurecr.io/clara-${svc}:latest
done
```

### Image Tagging

- `latest` — current production deployment
- `YYYYMMDD` — daily build tags for rollback
- Pin base image version in service Dockerfiles to prevent drift

## Deployment Runbook

### Phase 1: Infrastructure (One-Time)

1. Create resource group, Container Apps Environment, Key Vault, ACR
2. PostgreSQL Flexible Server (Burstable B1ms, 32GB storage, v16)
3. Azure Cache for Redis (Basic C0)
4. Azure Files shares for Qdrant and FalkorDB persistent data

### Phase 2: Data Services

1. Deploy Qdrant as Container App (internal ingress, 0.25 vCPU, 0.5Gi)
2. Deploy FalkorDB as Container App (internal ingress, 0.25 vCPU, 0.5Gi)
3. Validate persistent volume mounts

### Phase 3: Application Services

1. Deploy Gateway (internal ingress, 0.5 vCPU, 1Gi, min replicas 1)
2. Deploy Teams Adapter (external ingress on :3978, 0.25 vCPU, 0.5Gi, min replicas 1)
3. Deploy Backup (internal, 0.25 vCPU, 0.5Gi, min replicas 0)
4. Deploy Identity (internal, 0.25 vCPU, 0.5Gi, min replicas 0)

### Phase 4: Bot Framework

1. Get Teams Adapter external FQDN
2. Update Azure Bot Service messaging endpoint to `https://<FQDN>/api/messages`
3. Test bot response

## Configuration Management

### Key Vault Secrets Required

| Secret Name | Source | Used By |
|---|---|---|
| `teams-app-id` | Entra ID app registration | Teams Adapter |
| `teams-app-password` | Entra ID app registration | Teams Adapter |
| `clara-gateway-secret` | Generated | Gateway, Adapters |
| `postgres-connection-string` | PostgreSQL Flexible Server | Gateway |
| `redis-connection-string` | Redis Cache | Gateway |
| `openrouter-api-key` | OpenRouter dashboard | Gateway |
| `openai-api-key` | OpenAI dashboard | Gateway |
| `hf-token` | HuggingFace | Gateway |

### Injection Pattern

Container Apps reference Key Vault secrets via managed identity:
```
--secrets secret-name=keyvaultref:https://mypalclara-kv.vault.azure.net/secrets/secret-name,identityref:<managed-identity-id>
--env-vars ENV_VAR=secretref:secret-name
```

## Monitoring

### Health Checks

| Service | Check | Expected |
|---|---|---|
| Teams Adapter | `GET /api/health` | HTTP 200 |
| Gateway | WebSocket connect :18789 | Connection established |
| PostgreSQL | `pg_isready` | Ready |
| Redis | `PING` | PONG |

### Log Collection

- Container Apps → Log Analytics (automatic)
- Key metrics: response times, restart count, CPU/memory, error rates
- Critical alert: Teams Adapter 5xx > 5 in 5 min

### Update Process

```bash
# Build new image
docker build -f services/gateway/Dockerfile -t mypalclara.azurecr.io/clara-gateway:YYYYMMDD .
docker push mypalclara.azurecr.io/clara-gateway:YYYYMMDD

# Update Container App
az containerapp update --name gateway --resource-group mypalclara \
  --image mypalclara.azurecr.io/clara-gateway:YYYYMMDD

# Rollback if needed
az containerapp revision activate --resource-group mypalclara \
  --name gateway --revision <previous-revision>
```

## Risks

| Risk | Severity | Mitigation |
|---|---|---|
| Teams Adapter cold start > 15s Bot timeout | High | min replicas = 1 |
| Azure Files mount failures for Qdrant/FalkorDB | High | Test before data load |
| Base image drift | Medium | Pin version tags |
| Key Vault secret rotation | Medium | Document restart procedure |

## Dependencies

- **Platform**: Infrastructure provisioning
- **Security**: Entra ID app registration, Key Vault access policies
- **Integration**: Bot Service resource, Teams manifest sideloading

---

**Status**: Assessment complete. Manual deployment strategy ready.

---