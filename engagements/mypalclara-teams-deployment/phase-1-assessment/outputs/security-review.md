# Security Review: MyPalClara Teams Deployment

**Engagement**: mypalclara-teams-deployment
**Reviewer**: Security & Compliance Analyst
**Date**: 2026-04-20
**Status**: **CONDITIONALLY APPROVED**

---

## Executive Summary

MyPalClara deployment presents moderate security risks requiring specific controls before production readiness. Critical findings: secrets management modernization and data classification implementation required.

## SOC 2 Controls Assessment

### Trust Service Criteria: Security (Mandatory)

| Control Area | Status | Implementation Required |
|---|---|---|
| **CC6.1** - Access Management | ⚠️ HIGH | Managed identity for all Azure services, RBAC scoping |
| **CC6.2** - Authentication | ✅ COMPLIANT | Entra ID integration via Bot Framework |
| **CC6.3** - Authorization | ⚠️ HIGH | Least privilege RBAC, no subscription-level permissions |
| **CC6.6** - Logical Access | ⚠️ MEDIUM | Private endpoints for production databases |
| **CC6.7** - Transmission Security | ⚠️ HIGH | TLS 1.2+ enforcement, encrypted internal communication |
| **CC6.8** - System Monitoring | ⚠️ MEDIUM | Comprehensive logging to Log Analytics |

## High Severity Findings (Must fix before deployment)

### H-1: External API Secrets Management
**Finding**: Multiple external API keys (OpenRouter, Anthropic, OpenAI, HuggingFace) stored as configuration
**Recommendation**:
- Store all external API keys in Azure Key Vault
- Use Key Vault references in Container Apps configuration
- System-assigned managed identity for Container Apps → Key Vault access
- Implement 90-day rotation policy

### H-2: Database Connection Security
**Finding**: PostgreSQL connection strings contain credentials
**Recommendation**:
- Store connection strings in Key Vault
- Enable Entra ID authentication on PostgreSQL Flexible Server where possible
- Eliminate password-based database authentication where feasible

### H-3: TLS Configuration Enforcement
**Finding**: No explicit TLS 1.2+ enforcement documented
**Recommendation**:
- Minimum TLS 1.2 on all Container Apps ingress
- HTTPS only on PostgreSQL Flexible Server
- Verify all external API calls use TLS 1.2+

## Medium Severity Findings (Fix within 30 days)

### M-1: Network Segmentation
- Document intended communication flows between services
- Consider network policies within Container Apps environment

### M-2: Data Classification
- Chat messages: **Confidential** (PII risk)
- Memory embeddings: **Internal** (derived data)
- Implement data purging capability for user data requests

### M-3: Audit Logging
- Enable diagnostic settings on all Azure resources
- Send to Log Analytics workspace
- 12+ month retention for SOC 2
- Alert on suspicious access patterns

## Low Severity Findings (Fix within 90 days)

### L-1: Bot Framework credential rotation — migrate to certificate-based auth
### L-2: Container image scanning — enable in Container Registry

## Identity Security

### Approved
- Entra ID App Registration (multi-tenant) for Bot Framework
- System-assigned managed identities for Container Apps

### Required
- Key Vault Secrets User role (not Administrator) for managed identities
- Sign-in log monitoring for app registration

## Network Security

### Required Controls
1. Private endpoints for PostgreSQL Flexible Server (if cost-effective)
2. TLS enforcement on all service connections
3. Teams Adapter is only externally-exposed endpoint

### POC Acceptable Risks
- No DDoS Protection Standard (cost)
- No WAF (single bot endpoint, limited attack surface)
- Platform-managed encryption keys (CMK for production)

## Data Security

| Data Type | Classification | Storage | Encryption |
|---|---|---|---|
| Chat Messages | Confidential | PostgreSQL | At rest (PMK) |
| Memory Embeddings | Internal | Qdrant | Azure Files encryption |
| Knowledge Graph | Internal | FalkorDB | Azure Files encryption |
| User Sessions | Confidential | PostgreSQL | At rest (PMK) |
| API Keys | Highly Confidential | **Must be in Key Vault** | Key Vault HSM |

## Compliance Checklist

- [ ] CC6.1: RBAC scoping (Key Vault access, resource group scope)
- [ ] CC6.3: Authorization matrix documented
- [ ] CC6.6: Private endpoints for managed databases
- [ ] CC6.7: TLS 1.2+ on all paths
- [ ] CC6.8: Audit logging to Log Analytics
- [ ] CC7.1: Security monitoring and alerting
- [ ] CC8.1: Data management lifecycle documented

## Decision

**CONDITIONALLY APPROVED** — Deploy after resolving HIGH findings:
1. Key Vault for all secrets
2. Managed identity access to Key Vault
3. TLS 1.2+ enforcement
4. Security architecture documented

Report back confirming file was written.