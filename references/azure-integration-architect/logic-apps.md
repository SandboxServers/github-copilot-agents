# Logic Apps

## Hosting Models

### Consumption vs Standard

| Aspect | Consumption | Standard |
|---|---|---|
| **Hosting** | Multi-tenant, serverless | Single-tenant (Azure Functions runtime) |
| **Workflows per resource** | 1 | Multiple workflows in one resource |
| **Pricing** | Per execution (actions + triggers) | Fixed hosting plan (WS1/WS2/WS3) |
| **VNet integration** | Limited | Full (VNet integration or ASE v3) |
| **Local development** | Portal only | VS Code + local runtime |
| **Deployment** | ARM template | CI/CD like Functions (zip deploy, containers) |
| **Stateful/Stateless** | Stateful only | Both stateful and stateless workflows |
| **Connectors** | Managed connectors (Standard/Enterprise) | Built-in + managed connectors |
| **Best for** | Simple integrations, low volume, prototyping | Production, high throughput, enterprise |

### Selection Guidance
- **Consumption**: Quick prototyping, low volume, no VNet needs. Per-execution pricing makes sense at low scale.
- **Standard**: Production enterprise workloads. Need VNet, multiple workflows, local dev, predictable pricing.
- **Cost trap**: Consumption gets expensive fast with loops — each action in each iteration counts. Standard's fixed plan wins at moderate-to-high volume.

## Connectors

- **400+ connectors** for Azure services, Office 365, Salesforce, SAP, databases, file systems, and more.
- **Built-in connectors**: HTTP, Schedule, Batch, Service Bus, Event Hubs — run in-process, fastest performance.
- **Managed connectors**: Run in Azure as shared service. Standard tier and Enterprise tier (higher throughput, SLA).
- **Custom connectors**: OpenAPI-based (Swagger). Wrap any REST API as a Logic Apps connector.
- **On-premises connectors**: Via on-premises data gateway. SQL Server, file system, SAP, SharePoint on-prem.

## Workflow Patterns

- **Sequential**: Actions run one after another.
- **Parallel branches**: Multiple paths execute simultaneously, join at a downstream action.
- **For-each loops**: Iterate over arrays. Configurable concurrency (1 = sequential, up to 50 parallel).
- **Until loops**: Repeat until condition met or count/timeout reached.
- **Switch/case**: Route based on expression value.
- **Scope**: Group related actions. Acts as try-catch block for error handling.

## Error Handling

### Retry Policies
- **Default**: Exponential backoff, up to 4 retries, 7.5s intervals capped 5-45s.
- **Fixed interval**: Consistent delay between retries.
- **Exponential interval**: Increasing delay with configurable base and max.
- **None**: No retries. Fail immediately.
- Each action in a loop can have its own retry policy — be deliberate per operation.

### Scopes as Try-Catch
- Group related actions in a Scope.
- Configure downstream actions with `runAfter` on "Failed" status to handle errors.
- Pattern: Scope (try) → Scope (catch) with runAfter: Failed → Scope (finally) with runAfter: Succeeded, Failed.
- **Terminate action**: Explicitly fail the workflow with custom error code and message.

### Run-After Configuration
- Set actions to run on: Succeeded, Failed, Skipped, or TimedOut from predecessor.
- Enables compensation logic, notifications on failure, and graceful degradation.

## Data Transformation

- **Liquid templates**: JSON-to-JSON, JSON-to-text, XML-to-JSON transformations. Full template language with iterations, conditionals, variables.
- **XSLT maps**: XML-to-XML for B2B/EDI scenarios. Stored in Integration Account.
- **Inline Code**: JavaScript snippets for simple transforms (Standard only or Consumption with Integration Account).
- **Compose action**: Construct JSON objects inline.
- **Parse JSON**: Validate and extract fields from JSON payloads with schema.
- **Custom connectors/Functions**: Call Azure Functions or custom APIs for complex transformation logic.

## B2B and EDI (Integration Account)

- **Integration Account**: Required for B2B scenarios. Stores schemas, maps, partners, agreements.
- **EDI protocols**: AS2, X12, EDIFACT for electronic data interchange.
- **Partners and agreements**: Define trading partner profiles and message exchange agreements.
- **BizTalk migration target**: Logic Apps Standard + Integration Account replaces BizTalk Server.
- Custom functoids from BizTalk → inline code in Logic Apps or Azure Functions.

## Best Practices

1. **Use scopes for error handling**: Don't rely on individual action retry alone. Group and catch.
2. **Implement idempotency**: Check before action (e.g., check if record exists before creating).
3. **Chunk large payloads**: Enable chunking for large message transfer. Use claim-check pattern for very large data.
4. **Use managed connectors over HTTP**: Better monitoring, retry handling, and authentication management.
5. **Use tracked properties**: Add custom properties to actions for querying in Log Analytics.
6. **Limit loop concurrency**: Unbounded parallelism can overwhelm downstream services.
7. **Diagnostic settings**: Send to Log Analytics for cross-workflow analysis and alerting.
8. **Resubmit failed runs**: Use run history (Consumption) or trigger history (Standard) to replay failures.
