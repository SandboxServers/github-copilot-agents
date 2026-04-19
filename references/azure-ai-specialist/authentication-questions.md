# Authentication & Discovery Questions

## Authentication — Always Managed Identity

```
Every AI service connection:
├─ Azure OpenAI → Managed identity + RBAC (Cognitive Services OpenAI User)
├─ AI Search → Managed identity + RBAC (Search Index Data Reader)
├─ Cosmos DB → Managed identity + RBAC (Cosmos DB Built-in Data Reader)
├─ AI Foundry → Managed identity + workspace-level RBAC
├─ Key Vault (for any remaining secrets) → Managed identity + RBAC
└─ NEVER: API keys in code, config files, or environment variables
```

Use `DefaultAzureCredential` in application code — it chains through managed identity, workload identity, and developer credentials automatically.

## Questions This Specialist Always Asks

### Discovery
- What problem are you actually trying to solve with AI? (Not "we want to add AI" — what specific capability?)
- What data do you have available? (Format, volume, sensitivity, freshness requirements)
- What are the latency requirements? (Real-time chat vs batch processing vs near-real-time)
- What's the expected volume? (Tokens/day, requests/minute — this drives deployment type and cost)
- What compliance requirements apply? (Data residency, content filtering requirements, industry regulations)

### Architecture
- Do you need RAG? What's the knowledge source? (Documents, databases, APIs, SharePoint)
- Is this a single-turn or multi-turn interaction? (Affects context window management and cost)
- Do you need function calling / tool use? (Drives SDK choice and agent architecture)
- Is this user-facing or system-to-system? (Affects latency requirements, error handling, content filtering)
- What's the fallback if AI is unavailable? (Circuit breaker patterns, graceful degradation)

### Cost & Operations
- What's the monthly budget for AI services? (Be specific — this determines model and deployment type)
- Who monitors this in production? (Token usage, content filter triggers, latency, errors)
- How do you evaluate quality? (Ground truth datasets, human evaluation, automated evals)
- What's your model update strategy? (Pin versions vs auto-update, testing process for new versions)
- Have you considered batch where applicable? (50% discount for non-real-time workloads)
