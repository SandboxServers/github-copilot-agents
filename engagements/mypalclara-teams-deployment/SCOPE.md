# Engagement Scope: MyPalClara Teams Deployment

**Engagement Name**: mypalclara-teams-deployment
**Date**: 2026-04-20
**Requestor**: Steven Cady
**Status**: Planning Phase

---

## User Request

Deploy the MyPalClara AI chatbot application to Azure and integrate it as a Microsoft Teams chatbot. Use the Visual Studio Enterprise subscription. Contain all resources in a single `mypalclara` resource group. Use a region where all resources can be deployed together (target: West US 3 or Central US regions).

## Application Profile

MyPalClara is a multi-service AI chatbot with persistent memory, built in Python 3.12 (FastAPI/aiohttp), with a React + Rails web UI. Currently deployed on Railway.

### Services to Deploy

| Service | Technology | Port | Purpose |
|---|---|---|---|
| Gateway | Python 3.12 / FastAPI + Uvicorn | 18789 (WS), 18790 (HTTP) | Core message processing, LLM orchestration, tool execution |
| Teams Adapter | Python 3.12 / aiohttp + Bot Framework SDK | 3978 | Microsoft Teams Bot Framework endpoint |
| PostgreSQL 16 | Database | 5432 | Main database (sessions, messages, models) |
| Qdrant | Vector DB | 6333, 6334 | Semantic memory embeddings (1024-dim BGE-large) |
| Redis 7 | Cache | 6379 | Memory embedding cache |
| FalkorDB | Graph DB | 6379 | Knowledge graph (typed entities, temporal relationships) |
| Backup | Python 3.12 | 8080 | Database backup service |
| Identity | Python 3.12 / FastAPI | 18791 | OAuth/JWT identity service |

### External Dependencies

| Service | Purpose | Required |
|---|---|---|
| LLM Provider (OpenRouter/Anthropic/Azure OpenAI) | Chat completions | Yes |
| OpenAI API or HuggingFace | Embeddings (BGE-large-en-v1.5, 1024-dim) | Yes |
| Tavily | Web search tool | Optional |

### Teams Integration (Existing)

The application already has a full Teams adapter:
- `mypalclara/adapters/teams/` — Bot Framework SDK integration (botbuilder-core 4.16)
- `teams_manifest/manifest.json` — Complete Teams app manifest (v1.16) with bot commands (help, status, clear, memory)
- RSC permissions supported (no tenant-wide admin consent needed)
- Requires: Azure Bot Resource + Entra ID App Registration
- Messaging endpoint: `https://<domain>/api/messages` on port 3978

### Docker Architecture

All services use a shared base image (`clara-base`) built from `services/base/Dockerfile` (Python 3.12-slim). Individual services add their entrypoint. Docker Compose profiles control which services run:
- `gateway` profile: Gateway service
- `teams` profile: Teams adapter
- Default: PostgreSQL, Qdrant, Redis, FalkorDB, Backup

---

## Constraints

1. **Subscription**: Visual Studio Enterprise Subscription (~$150/month credits)
2. **Resource Group**: Single `mypalclara` resource group
3. **Region**: All resources in one region. Prefer West US 3 or Central US. Check quotas.
4. **SOC2 Readiness**: Encryption at rest/transit, audit logging, access controls, secrets management
5. **Cost Baseline**: Established and documented
6. **Documentation**: Complete for handoff and reimplementation in other tenants
7. **License**: PolyForm Noncommercial 1.0.0 — personal/internal use only

---

## Success Criteria

1. **Application running on Azure compute** — Gateway + Teams Adapter containers running, healthy, reachable
2. **Teams app manifest created and registered** — Entra ID app registration, Azure Bot Resource, manifest uploaded
3. **Teams bot endpoint configured** — Messaging endpoint points to Azure-hosted Teams adapter, TLS terminated
4. **Proof of concept: bot responds** — Clara responds to @mention in a test Teams channel or direct message

---

## Budget & Timeline

- **Budget**: Visual Studio Enterprise subscription credits (~$150/month). Minimize spend.
- **Timeline**: Single engagement, planning phase first, implementation follows plan approval.

---

## What's In Scope

- Azure infrastructure provisioning (compute, networking, databases, identity)
- Azure Bot Resource + Entra ID app registration
- Teams app manifest packaging and sideloading
- Container deployment (Docker images → Azure compute)
- Database provisioning (PostgreSQL, Redis, Qdrant, FalkorDB)
- Secrets management (Key Vault)
- Networking (TLS termination for bot endpoint)
- Monitoring baseline
- Cost estimate
- Security review
- Deployment documentation

## What's Out of Scope

- Web UI deployment (Rails + React) — Teams bot is the target interface
- Discord adapter deployment
- Voice/TTS features
- CI/CD pipeline (manual deployment for POC)
- Custom domain / DNS beyond Azure defaults
- Production HA / multi-region DR (this is a POC)
- Commercial licensing considerations