# Security & Compliance Analyst — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Security Frameworks | [security-frameworks.md](security-frameworks.md) | When referencing MCSB, CIS Controls, or NIST SP 800-53 |
| Identity Security | [identity-security.md](identity-security.md) | When reviewing identity posture — least privilege, service principals, managed identity |
| Network Security | [network-security.md](network-security.md) | When reviewing network exposure — endpoints, NSGs, TLS, WAF, DDoS |
| Data Protection | [data-protection.md](data-protection.md) | When reviewing encryption, data classification, PII handling, or key management |
| Secrets Management | [secrets-management.md](secrets-management.md) | When reviewing secrets handling, Key Vault usage, credential rotation |
| Defender for Cloud | [defender-for-cloud.md](defender-for-cloud.md) | When working with CSPM, Defender plans, Secure Score, or attack path analysis |
| Sentinel | [sentinel.md](sentinel.md) | When working with SIEM/SOAR, analytics rules, data connectors, or hunting |
| Azure Policy Security | [azure-policy-security.md](azure-policy-security.md) | When implementing security policies, compliance initiatives, or DINE remediation |
| Compliance Frameworks | [compliance-frameworks.md](compliance-frameworks.md) | When evaluating SOC 2, HIPAA, PCI-DSS, or FedRAMP requirements |
| Threat Modeling | [threat-modeling.md](threat-modeling.md) | When performing STRIDE analysis or architecture attack surface review |
| Security Review Process | [security-review-process.md](security-review-process.md) | When producing security review documents or classifying findings |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When identifying common security mistakes — afterthought security, over-permissions, shared creds |
| Gotchas | [gotchas.md](gotchas.md) | When troubleshooting surprising behaviors — managed identity lifecycle, Key Vault soft delete, CA evaluation |
| Best Practices | [best-practices.md](best-practices.md) | When recommending security controls — Zero Trust, managed identity, private endpoints, RBAC |
| Security Operations Gaps | [security-operations-gaps.md](security-operations-gaps.md) | When running threat modeling sessions or mapping multi-framework compliance controls |

## Related Skills

| Skill | When to Load |
|-------|-------------|
| `security-review-framework` | Full security review checklist and threat modeling |
| `secrets-management-audit` | Auth hierarchy, Key Vault config, rotation, detection |
| `network-security-design` | Private endpoints, NSGs, zero trust networking |
| `kubernetes-security-hardening` | AKS cluster, pod, network, and image security |
