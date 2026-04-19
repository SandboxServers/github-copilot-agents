## Hybrid Integration

### On-Premises Data Gateway
- Bridge for Logic Apps to access on-prem systems (SQL Server, file systems, SAP, BizTalk)
- Install gateway agent on on-prem Windows server → registers with Azure → Logic Apps uses gateway connection
- **Gotcha**: single point of failure unless clustered. Plan for HA with gateway cluster mode.

### Self-Hosted Gateway (APIM)
- Run APIM gateway on your own infrastructure (Kubernetes, Docker)
- Available in Developer and Premium tiers
- **Use case**: low-latency access to on-prem backends, compliance requiring local processing
- Syncs configuration from APIM management plane

### BizTalk Migration
- Logic Apps is the modern replacement for BizTalk Server
- Integration accounts provide B2B capabilities: AS2, X12, EDIFACT
- **Migration path**: BizTalk → Logic Apps Standard + Integration Account
- EDI schemas, maps, and partner agreements port to integration accounts
- **Gotcha**: not all BizTalk adapters have direct Logic Apps connector equivalents — may need custom connectors or Azure Functions as bridges
