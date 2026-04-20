## Platform Engineering Capability Model

### Purpose
The capability model provides a structured way to assess, plan, and improve
platform engineering maturity. Each capability represents a domain that the
platform must address to serve development teams effectively.

### Core Capabilities

#### 1. Service Catalog
A discoverable inventory of all platform services, tools, and resources.
- What services are available and how to consume them
- Version information, SLAs, and ownership
- Links to documentation and support channels

#### 2. Golden Paths
Opinionated, pre-configured workflows for common development scenarios.
- New service creation (repo + CI/CD + infra + monitoring)
- Standard deployment patterns
- Database provisioning with backup and security defaults

#### 3. Self-Service Provisioning
Developers request and receive resources without filing tickets.
- Infrastructure provisioning through portal, CLI, or API
- Environment creation (dev, staging, production)
- Access management within policy boundaries

#### 4. Observability
Consistent monitoring, logging, and alerting across all platform services.
- Centralized logging with standard formats
- Application performance monitoring
- Health dashboards per service and per team
- Alerting with meaningful thresholds and routing

#### 5. Security Guardrails
Security controls embedded in the platform rather than bolted on.
- Policy-as-code enforcement (Azure Policy, OPA)
- Secret management and rotation
- Vulnerability scanning in CI/CD pipelines
- Network segmentation by default

#### 6. Cost Visibility
Teams can see and manage their cloud spend.
- Cost attribution by team, service, and environment
- Budget alerts and spending trends
- Approved SKU lists to prevent overspending
- Chargeback or showback reporting

#### 7. Documentation
Comprehensive, up-to-date documentation for all platform capabilities.
- Getting started guides for each golden path
- API references for platform services
- Architecture decision records
- Runbooks for common operations

#### 8. Developer Onboarding
Streamlined process for new developers and new teams.
- Day-one environment setup automation
- Access provisioning for tools and services
- Training materials and self-paced learning paths
- Mentorship and office hours

### Maturity Levels

| Level | Name | Description |
|---|---|---|
| 1 | **Ad-hoc** | No standard approach. Teams solve problems independently with inconsistent tooling and processes. |
| 2 | **Defined** | Standard tools and processes documented. Requires domain expertise to execute. Manual steps remain. |
| 3 | **Measured** | Self-service available. Adoption and satisfaction tracked. Feedback loops operational. |
| 4 | **Optimized** | Data-driven continuous improvement. Automated adaptation. High adoption with minimal support burden. |

### Assessing Maturity Per Capability
Rate each capability independently. Most organizations have uneven maturity:

| Capability | Typical Starting Point |
|---|---|
| Service Catalog | Ad-hoc (tribal knowledge, no central inventory) |
| Golden Paths | Ad-hoc to Defined (some templates exist but aren't maintained) |
| Self-Service Provisioning | Ad-hoc (ticket-based, manual fulfillment) |
| Observability | Defined (tools exist but inconsistently adopted) |
| Security Guardrails | Defined (policies written but not all enforced) |
| Cost Visibility | Ad-hoc (finance reports monthly, no team attribution) |
| Documentation | Ad-hoc (outdated wikis, tribal knowledge) |
| Developer Onboarding | Ad-hoc (varies by team, lengthy manual process) |

### Prioritizing Improvement
- Focus on capabilities where improvement delivers the most value to the most teams
- Advance capabilities to Defined before pursuing Measured or Optimized
- Validate adoption at each level before investing in the next
- A capability at Measured that nobody uses is worse than Ad-hoc that everyone workarounds
