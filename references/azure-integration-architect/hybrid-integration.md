# Hybrid Integration

## On-Premises Connectivity Options

### On-Premises Data Gateway
- Bridge for Logic Apps, Power Platform, and Data Factory to access on-prem systems.
- Supported systems: SQL Server, file systems, SAP, BizTalk, Oracle, DB2, Informix, SharePoint.
- Install gateway agent on on-prem Windows server → registers with Azure → cloud services use gateway connection.
- **No inbound firewall rules needed**: Gateway initiates outbound connection to Azure Service Bus relay.
- **High availability**: Gateway cluster mode — install multiple agents for failover. Required for production.
- **CRITICAL**: Single gateway agent is a single point of failure. Always cluster for production workloads.

### Azure Relay
- **Hybrid Connections**: TCP and HTTP tunneling to on-prem without VPN. No code changes needed for HTTP.
- **WCF Relay**: Legacy SOAP/WCF service exposure to cloud. For modernizing existing WCF services.
- Use when: Need specific endpoint connectivity without full network integration. Lighter than VPN.
- Hybrid Connections used by App Service, Functions for on-prem database/API access.

### VPN Gateway / ExpressRoute
- **VPN Gateway**: IPsec tunnel over internet. Site-to-site or point-to-site. Encrypted, cost-effective.
- **ExpressRoute**: Private connection via connectivity provider. Higher bandwidth, lower latency, more reliable.
- Use with private endpoints for Azure services. Full network-level connectivity.
- Required for scenarios needing broad network access (not just specific endpoints).

### Self-Hosted Integration Runtime (Data Factory / Synapse)
- Agent installed on-prem for Data Factory and Synapse pipelines to access on-prem data sources.
- Supports: SQL Server, Oracle, file systems, HDFS, and many more on-prem data stores.
- Install agent → register with Data Factory → use in pipeline linked services.
- HA mode: Install on multiple machines with shared registration.

### APIM Self-Hosted Gateway
- Run API Management gateway container on your own infrastructure (Kubernetes, Docker, Arc-enabled K8s).
- Available in Developer and Premium tiers.
- **Use cases**: Low-latency access to on-prem backends, compliance requiring local processing, edge deployments.
- Syncs configuration from APIM management plane in Azure. Operates independently if connection temporarily lost.
- Consistent policies, analytics, and management across cloud and on-prem gateways.

## BizTalk Server Migration

### Migration Target: Logic Apps Standard + Integration Account

| BizTalk Component | Azure Equivalent |
|---|---|
| Orchestrations | Logic Apps workflows (stateful) |
| Send/Receive ports | Logic Apps connectors + triggers |
| Maps (XSLT) | Integration Account maps |
| Schemas (XSD) | Integration Account schemas |
| Pipelines | Logic Apps built-in actions |
| Business Rules Engine | Logic Apps conditions + Azure Functions |
| EDI (AS2, X12, EDIFACT) | Integration Account agreements |
| BAM (monitoring) | Application Insights + Log Analytics |
| Custom functoids | Inline code or Azure Functions |
| Adapters | Logic Apps connectors (400+) |

### Migration Gotchas
- Not all BizTalk adapters have direct Logic Apps connector equivalents — may need custom connectors or Azure Functions as bridges.
- Complex orchestrations with correlation sets → Logic Apps sessions or Durable Functions.
- BizTalk's in-order delivery via sequential convoys → Service Bus sessions.
- Performance characteristics differ — load test before migration.

## SAP Integration

### Logic Apps SAP Connector
- RFC (Remote Function Call), BAPI (Business API), IDoc (Intermediate Document) support.
- Requires SAP .NET Connector (NCo) library and on-premises data gateway.
- Send and receive IDocs, call RFCs, trigger on SAP events.

### Other SAP Options
- **APIM SAP OData connector**: Expose SAP OData services as REST APIs through APIM.
- **Data Factory SAP connectors**: SAP Table, SAP ECC, SAP BW, SAP HANA for data extraction.
- **Azure Integration Services for SAP**: Comprehensive SAP integration using Logic Apps + APIM + Service Bus.

## Hybrid Architecture Patterns

### Pattern 1: API-Led Connectivity
```
External Consumers → APIM (cloud) → APIM Self-Hosted Gateway (on-prem) → On-Prem APIs
```
Consistent API management across locations.

### Pattern 2: Event Bridge
```
On-Prem System → Service Bus (hybrid connection) → Event Grid → Cloud Consumers
```
On-prem events routed to cloud subscribers.

### Pattern 3: Data Integration
```
On-Prem Database → Self-Hosted IR → Data Factory → Azure Data Lake / Synapse
```
Batch data movement from on-prem to cloud analytics.
