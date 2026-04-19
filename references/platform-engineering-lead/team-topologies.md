## Team Topologies

### Platform Team Responsibilities
From the Cloud Adoption Framework DevOps team topologies guidance:

| Category | Responsibilities |
|---|---|
| **Governance & Compliance** | Architecture governance, policy management and enforcement, security monitoring and audits |
| **Resource Provisioning** | Subscription provisioning and delegation, platform as code (templates, scripts, modules) |
| **Identity & Access** | Identity and access management policies, Azure RBAC, role definitions |
| **Network Management** | Network management, delegation of network policies |
| **Operations** | Overall operations on Azure, management of service principals, Microsoft Graph API registration |
| **Key Management** | Key management for central services |
| **Platform Observability** | Platform management and monitoring, cost management |

### Three Team Types
| Team Type | Role |
|---|---|
| **Platform teams** | Build internal platforms to accelerate delivery and reduce cognitive load. Own the Azure Landing Zone lifecycle. |
| **Application workload teams** | Build applications that drive business outcomes. Own the end-to-end lifecycle of workloads in landing zones. |
| **Enabling teams** | Help overcome skill gaps by assisting other teams with specialized capabilities like DevOps. |

### Cross-Functional Composition
Platform teams should include members from IT, security, compliance, and business units. This:
- Ensures the platform addresses technical, security, compliance, and business needs simultaneously
- Prevents silos
- Accelerates decision-making (team resolves conflicts internally rather than through approval chains)
