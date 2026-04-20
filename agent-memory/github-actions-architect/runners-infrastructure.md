# Runners and Infrastructure — Deep Reference

## GitHub-Hosted Runners

### Standard Runner Labels

| Label | OS | Architecture | Notes |
|-------|-----|-------------|-------|
| `ubuntu-latest` | Ubuntu 24.04 | x64 | Default Linux choice |
| `ubuntu-24.04` | Ubuntu 24.04 | x64 | Explicit version |
| `ubuntu-22.04` | Ubuntu 22.04 | x64 | Previous LTS |
| `windows-latest` | Windows Server 2025 | x64 | Default Windows choice |
| `windows-2022` | Windows Server 2022 | x64 | Previous version |
| `macos-latest` | macOS 15 Sequoia | ARM64 (M1) | Apple Silicon |
| `macos-15` | macOS 15 Sequoia | ARM64 (M1) | Explicit version |
| `macos-14` | macOS 14 Sonoma | ARM64 (M1) | Previous version |
| `macos-13` | macOS 13 Ventura | x64 (Intel) | Last Intel macOS |

Standard runners: **4 vCPU / 16 GB RAM** (Linux/Windows), **3-core / 7 GB RAM** (macOS M1).

### Larger Runners (Team / Enterprise Cloud Plans)

| Configuration | vCPU | RAM | Notes |
|--------------|------|-----|-------|
| Linux 2–64 core | 2–64 | 8–256 GB | Custom labels |
| Windows 2–64 core | 2–64 | 8–256 GB | Custom labels |
| macOS M1 (3/6/12 core) | 3/6/12 | 7/14/30 GB | Apple Silicon |
| macOS M2 (3/6/12 core) | 3/6/12 | 7/14/30 GB | Newer Apple Silicon |
| GPU runners (Linux) | 4 | 28 GB | NVIDIA T4 (16 GB VRAM) |

Larger runners support **custom labels** and **runner groups** for targeting:

```yaml
jobs:
  heavy-build:
    runs-on: my-org-linux-16core  # custom label for 16-core larger runner
    steps:
      - uses: actions/checkout@v4
      - run: make build-all
```

### ARM Runners

```yaml
jobs:
  arm-build:
    runs-on: ubuntu-latest-arm64   # ARM64 runner
    steps:
      - uses: actions/checkout@v4
      - run: uname -m  # outputs aarch64
```

ARM runners are cost-efficient for ARM-native builds (Docker ARM images, Go ARM binaries).

## Self-Hosted Runners

### Registration

```bash
# Download and configure the runner
./config.sh --url https://github.com/my-org/my-repo \
  --token <REGISTRATION_TOKEN> \
  --name "build-server-01" \
  --labels "self-hosted,linux,x64,gpu" \
  --work "_work" \
  --runasservice
```

### Custom Labels

Self-hosted runners can have multiple labels. Jobs match using `runs-on`:

```yaml
jobs:
  gpu-job:
    runs-on: [self-hosted, linux, gpu]   # must match ALL labels
    steps:
      - run: nvidia-smi

  windows-build:
    runs-on: [self-hosted, windows, x64]
    steps:
      - run: msbuild MySolution.sln
```

## Ephemeral / Just-in-Time (JIT) Runners

Ephemeral runners execute **one job** then auto-deregister. Recommended for security.

```bash
# Configure as ephemeral
./config.sh --url https://github.com/my-org \
  --token <TOKEN> \
  --ephemeral \
  --name "ephemeral-$(uuidgen)"
```

JIT runners via REST API:

```bash
# Create a JIT runner config (organization level)
curl -X POST \
  -H "Authorization: Bearer $TOKEN" \
  https://api.github.com/orgs/my-org/actions/runners/generate-jitconfig \
  -d '{
    "name": "jit-runner-001",
    "runner_group_id": 1,
    "labels": ["self-hosted", "linux", "x64"],
    "work_folder": "_work"
  }'
# Returns encoded JIT config — pass to runner at startup
```

Benefits:
- No persistent state between jobs — clean environment every time
- No long-lived credentials on the runner machine
- Prevents lateral movement between jobs

## Runner Context Variables

Available in expressions and `run:` scripts:

| Context | Value |
|---------|-------|
| `runner.os` | `Linux`, `Windows`, `macOS` |
| `runner.arch` | `X64`, `X86`, `ARM`, `ARM64` |
| `runner.environment` | `github-hosted` or `self-hosted` |
| `runner.debug` | `1` if debug logging enabled, empty otherwise |
| `runner.temp` | Path to temp directory (cleaned after each job) |
| `runner.tool_cache` | Path to tool cache directory |
| `runner.name` | Name of the runner executing the job |

```yaml
- name: Platform-specific build
  run: |
    echo "OS: ${{ runner.os }}"
    echo "Arch: ${{ runner.arch }}"
    echo "Temp: ${{ runner.temp }}"
```

## Scaling Self-Hosted Runners

### Actions Runner Controller (ARC) on Kubernetes

ARC is GitHub's official solution for auto-scaling runners on Kubernetes:

```yaml
# helm install for ARC
# values.yaml
githubConfigUrl: "https://github.com/my-org"
githubConfigSecret:
  github_app_id: "12345"
  github_app_installation_id: "67890"
  github_app_private_key: |
    -----BEGIN RSA PRIVATE KEY-----
    ...
    -----END RSA PRIVATE KEY-----
minRunners: 1
maxRunners: 20
```

```yaml
# Reference ARC runners in workflows
jobs:
  build:
    runs-on: arc-runner-set   # matches the RunnerScaleSet name
```

### Auto-Scaling Patterns

| Pattern | Mechanism | Best For |
|---------|-----------|----------|
| ARC (Kubernetes) | Pod-per-job auto-scaling | Orgs with K8s infrastructure |
| VMSS (Azure) | VM scale set with webhook triggers | Azure-native workloads |
| Philips/terraform-aws-github-runner | Ephemeral EC2 instances | AWS-native workloads |
| Webhook-based | Listen for `workflow_job` events | Custom infrastructure |

## Runner Security Best Practices

1. **Always use ephemeral mode** for self-hosted runners — wipe after each job
2. **Never store long-lived credentials** on runner machines
3. **Isolate runner networks** — don't give runners access to production databases
4. **Use runner groups** to restrict which repos can use sensitive runners
5. **Never use self-hosted runners for public repos** — anyone can submit a PR that runs on your infrastructure
6. **Monitor runner activity** — audit logs for unexpected job executions
7. **Use dedicated VMs/containers** — don't run self-hosted runners on shared workstations

## Cost Optimization

| Strategy | When |
|----------|------|
| Self-hosted runners | High-volume repos (>2000 min/month) — GitHub-hosted billed per-minute |
| Larger runners | When parallel build saves more developer time than runner cost |
| ARM runners | Linux builds that don't need x64 — lower per-minute cost |
| Ephemeral spot/preemptible | Non-critical jobs (linting, formatting) that can tolerate eviction |
| Caching | Reduce job duration across all runner types |
| Path filtering | Don't run CI on docs-only changes |

## Pre-Installed Software

Each GitHub-hosted runner image includes hundreds of pre-installed tools. Check what's available:

```yaml
- name: Check installed tools
  run: |
    node --version
    python3 --version
    dotnet --list-sdks
    docker --version
    az --version
    terraform --version
```

Full lists maintained at:
- [actions/runner-images](https://github.com/actions/runner-images) — Ubuntu, Windows, macOS (GitHub-owned)
- [actions/partner-runner-images](https://github.com/actions/partner-runner-images) — ARM64 images

## Runner Groups (Organization-Level)

Runner groups restrict which repos and workflows can use specific runners:

```text
Runner Group: "production-deployers"
├── Allowed repositories: my-app, my-api (not all repos)
├── Allowed workflows: deploy.yml only (optional further restriction)
└── Runners: prod-deploy-01, prod-deploy-02
```

Configure via Organization Settings → Actions → Runner groups.

## Service Containers

Service containers provide databases and services for integration testing. **Linux runners only**.

```yaml
jobs:
  integration-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4
      - run: |
          # Services accessible via localhost when using port mapping
          psql -h localhost -U postgres -d testdb -c "SELECT 1"
          redis-cli -h localhost ping
        env:
          PGPASSWORD: testpass
```

### Container Jobs (Docker Network)

When using `container:` for the job itself, services share a Docker network — reference by service name instead of `localhost`:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    container: node:20
    services:
      db:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: testpass
    steps:
      - run: |
          # Reference by service name in container jobs
          psql -h db -U postgres -c "SELECT 1"
```
