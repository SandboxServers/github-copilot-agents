# Teams Administrator — Knowledge Index

> **Last Updated:** 2026-04-19 — Added anti-patterns, gotchas, best-practices, gap analysis.

Load only the files relevant to your current task. Do NOT load all files at once.

| Topic | File | When to Load |
|-------|------|-------------|
| Teams Administration | [teams-administration.md](teams-administration.md) | When working with Teams admin center, policy types, or policy inheritance |
| Meeting Policies | [meeting-policies.md](meeting-policies.md) | When configuring meeting settings, lobby, recording, or Copilot policies |
| Calling Policies | [calling-policies.md](calling-policies.md) | When designing PSTN — Direct Routing, Operator Connect, Calling Plans, AA/CQ |
| App Management | [app-management.md](app-management.md) | When managing Teams apps, permissions, and app setup policies |
| Compliance | [compliance.md](compliance.md) | When implementing retention, DLP, eDiscovery, or communication compliance |
| Guest & External Access | [guest-external-access.md](guest-external-access.md) | When configuring guest policies, external federation, or anonymous access |
| Teams Rooms | [teams-rooms.md](teams-rooms.md) | When managing Teams Rooms, Surface Hubs, or device policies |
| Call Quality | [call-quality.md](call-quality.md) | When diagnosing call quality with CQD, Call Analytics, or QoS |
| Teams Governance | [teams-governance.md](teams-governance.md) | When managing team lifecycle — creation, naming, expiration, archival |
| Anti-Patterns | [anti-patterns.md](anti-patterns.md) | When reviewing or auditing Teams configurations for common misconfigurations |
| Gotchas | [gotchas.md](gotchas.md) | When troubleshooting unexpected behavior — policy hierarchy, licensing quirks, storage locations |
| Best Practices | [best-practices.md](best-practices.md) | When designing governance, security, voice, quality, compliance, or device management strategies |

## Gap Analysis

Existing domain-specific references (teams-administration, meeting-policies, calling-policies, app-management, compliance, guest-external-access, teams-rooms, call-quality, teams-governance) provide strong topical depth. The new cross-cutting files fill the following gaps:

- **anti-patterns.md** — Consolidates common misconfigurations scattered across multiple domain files into a single audit checklist; adds coverage for Teams proliferation, emergency calling, and update management not covered elsewhere.
- **gotchas.md** — Captures non-obvious platform behaviors (policy hierarchy precedence, recording storage model, retroactive app policy limitations, voicemail Exchange dependency) that don't fit neatly into any single domain file.
- **best-practices.md** — Provides a unified operational playbook spanning governance, security, voice, quality, compliance, apps, and devices; serves as a starting-point reference when designing net-new Teams environments.

**Remaining potential gaps** (candidates for future additions):
- Teams Premium feature matrix and licensing decision tree
- Teams Phone migration playbook (PBX-to-Teams cutover planning)
- Teams for frontline workers (Shifts, Walkie Talkie, task publishing)
- Copilot in Teams configuration and data governance
