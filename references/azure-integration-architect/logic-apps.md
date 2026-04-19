## Logic Apps

### Standard vs Consumption
| Aspect | Consumption | Standard |
|---|---|---|
| **Hosting** | Multitenant | Single-tenant (Azure Functions runtime) |
| **Workflows per resource** | 1 | Multiple |
| **Pricing** | Per execution (actions + triggers) | Fixed hosting plan (like App Service) |
| **VNet integration** | Limited | Full (via ASE v3 or VNet integration) |
| **Local development** | Portal only | VS Code + local runtime |
| **Deployment** | ARM template | CI/CD like Functions (zip deploy, containers) |
| **Portability** | Azure only | Can run on ASE v3, hybrid with own infra |
| **Connectors** | Managed connectors (Standard/Enterprise/ISE) | Built-in + managed connectors |
| **Stateful/Stateless** | Stateful only | Both stateful and stateless workflows |
| **Best for** | Simple integrations, low volume | Production workloads, high throughput, enterprise |

### Key Decision Points
- **Consumption**: quick prototyping, low volume, no VNet needs, per-execution pricing makes sense at low scale
- **Standard**: production enterprise workloads, need VNet, need multiple workflows in one resource, need local dev, predictable pricing at scale
- **Cost trap**: Consumption gets expensive fast with loops — each action in each iteration counts. Standard's fixed plan wins at moderate-to-high volume.

### Error Handling
- **Retry policies**: Default (exponential, up to 4 retries), Fixed interval, Exponential interval, None
- **Scopes**: group related actions — if any action in scope fails, run compensating logic in the scope's `runAfter` with "Failed" status
- **Configure-run-after**: set actions to run on Success, Failed, Skipped, or Timed Out from predecessor
- **Terminate action**: explicitly fail the workflow with custom error code and message
- **Each action in a loop can have its own retry policy** — be deliberate about retry behavior per operation

### Data Transformation
- **Liquid templates**: JSON-to-JSON, JSON-to-text, XML-to-JSON, XML-to-text transformations
  - Stored in integration account (Consumption) or directly in Standard logic app resource
  - Support iterations, conditionals, variables — full Liquid template language
- **XSLT maps**: XML-to-XML transformations for B2B/EDI scenarios
- **Inline Code**: run JavaScript snippets within workflow for simple transformations
- **Compose action**: construct JSON objects inline
- **Parse JSON**: validate and extract fields from JSON payloads
- **Custom connectors**: when built-in transformation isn't enough, call Azure Functions or custom APIs
