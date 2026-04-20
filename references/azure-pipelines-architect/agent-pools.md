# Azure Pipelines Agent Pools and Management

## Microsoft-Hosted Agents

Microsoft-hosted agents are provisioned fresh for every job and discarded afterward.

| Image | Label | OS | Key Software |
|-------|-------|----|-------------|
| Ubuntu 24.04 | `ubuntu-latest` | Linux | Docker, .NET, Node, Python, Java, Go, kubectl |
| Ubuntu 22.04 | `ubuntu-22.04` | Linux | Same as above, LTS |
| Windows Server 2022 | `windows-latest` | Windows | VS 2022, .NET Framework, Docker, MSBuild |
| Windows Server 2019 | `windows-2019` | Windows | VS 2019, .NET Framework |
| macOS 14 (Sonoma) | `macos-latest` | macOS | Xcode 15+, CocoaPods, Homebrew |
| macOS 13 (Ventura) | `macos-13` | macOS | Xcode 14+, CocoaPods |

**Limitations:**
- 10 GB disk space available for user content
- Free tier: 1 parallel job, 1800 minutes/month, 60-minute job timeout
- Paid tier: 6-hour max job timeout per job
- No persistent storage — every job starts clean
- Cannot SSH into the agent or install system-level dependencies permanently

```yaml
pool:
  vmImage: 'ubuntu-latest'    # Microsoft-hosted

# Override image per job
jobs:
  - job: BuildLinux
    pool:
      vmImage: 'ubuntu-latest'
  - job: BuildWindows
    pool:
      vmImage: 'windows-latest'
```

## Self-Hosted Agents

Self-hosted agents run on infrastructure you control — VMs, physical machines, or containers.

```yaml
pool:
  name: 'MyPrivatePool'          # Organization/project agent pool
  demands:
    - Agent.OS -equals Linux
    - docker                      # Capability must exist on agent
    - maven
```

**Setup checklist:**
1. Download agent from Organization Settings > Agent pools > New agent
2. Configure: `./config.sh --unattended --url https://dev.azure.com/myorg --auth pat --token <PAT> --pool MyPool --agent my-agent-01`
3. Run as a service: `./svc.sh install && ./svc.sh start`
4. Agents auto-register capabilities based on installed software (detected from PATH)
5. Add custom capabilities manually via Agent pools > Agents > Capabilities

## Agent Capabilities and Demands

Capabilities are key-value pairs advertising what an agent can do. Demands filter which agents can run a job.

```yaml
# System capabilities (auto-detected): Agent.OS, Agent.Version, docker, npm, java
# User capabilities (manually added): custom-tool, environment=staging

pool:
  name: SelfHostedPool
  demands:
    - custom-tool                     # Exists check (capability must be present)
    - environment -equals staging     # Value check
```

If no agent in the pool satisfies all demands, the job queues indefinitely until one becomes available or times out.

## Azure Virtual Machine Scale Set (VMSS) Agents

VMSS agents auto-scale based on pipeline demand — ideal for burst workloads.

```
Configuration (via Azure DevOps UI):
  Pool type: Azure virtual machine scale set
  Scale set: /subscriptions/.../virtualMachineScaleSets/my-vmss
  Max agents: 10
  Standby agents: 2             # Pre-warmed agents ready to pick up work
  Idle timeout: 15 minutes      # Time before idle agent is deallocated
  Grace period: 15 minutes      # Time before a newly provisioned agent is eligible for deallocation
```

**Key behaviors:**
- Azure DevOps manages scaling — do NOT configure Azure Autoscale on the VMSS
- Agents are reimaged after each job (by default) for clean state
- Standby count determines minimum warm agents (cost vs responsiveness tradeoff)
- Custom images: bake your tools into the VMSS image to avoid install time per job

## Elastic Agents with Kubernetes (KEDA)

For Kubernetes-based scaling, use the Azure Pipelines agent with KEDA (Kubernetes Event-Driven Autoscaling):

- Deploy agent containers to AKS or any Kubernetes cluster
- KEDA scaler monitors the Azure DevOps agent pool queue
- Scales from zero to N based on pending jobs
- Best for containerized build workloads with variable demand

## Parallel Jobs

| Tier | Free (public projects) | Free (private) | Purchased |
|------|----------------------|----------------|-----------|
| Microsoft-hosted | 10 parallel jobs | 1 job (1800 min/month) | $40/month per additional |
| Self-hosted | Unlimited | 1 parallel job | $15/month per additional |

**Estimating parallel jobs needed:**
- Calculate: `(avg pipeline duration in minutes) × (avg runs per day) / (available minutes per day)`
- A team running 50 builds/day averaging 10 minutes each needs ~50×10/480 ≈ 1 parallel job minimum
- Add headroom for peak hours and queue wait tolerance

## Container Jobs

Run jobs inside a container image on any agent (hosted or self-hosted):

```yaml
jobs:
  - job: BuildInContainer
    pool:
      vmImage: 'ubuntu-latest'
    container:
      image: mcr.microsoft.com/dotnet/sdk:8.0
      options: --user 0:0        # Run as root if needed
    steps:
      - script: dotnet build
```

**Docker-in-Docker considerations:**
- Microsoft-hosted agents support DinD natively
- Self-hosted: mount Docker socket or run Docker-in-Docker sidecar
- Security risk: container escape possible with Docker socket mount

## Maintenance Jobs

Schedule maintenance on self-hosted agents to prevent disk/state bloat:

- Configure via Agent pools > Settings > Maintenance
- Runs cleanup scripts (delete old working directories, prune Docker images)
- Schedule during off-hours to avoid blocking pipeline jobs

## Decision Matrix: Hosted vs Self-Hosted

| Factor | Microsoft-Hosted | Self-Hosted |
|--------|-----------------|-------------|
| Maintenance | Zero | You manage updates/patches |
| Build speed | Clean VM each job (cold start) | Warm cache, pre-installed tools |
| Cost model | Per-minute / per-parallel-job | Infrastructure + parallel job license |
| Network access | Public internet only | Access private networks/VPNs |
| Custom hardware | No (standard VMs) | GPU, ARM, specialized hardware |
| Compliance | Microsoft-managed | Full control over data residency |
| Scale | Automatic | VMSS or manual |
| Recommended for | Standard builds, OSS projects | Enterprise, air-gapped, GPU/ML |
