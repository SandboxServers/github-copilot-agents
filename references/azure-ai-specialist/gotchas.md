# Azure AI Gotchas

Surprising behaviors and non-obvious details that catch teams deploying Azure AI services.

## Token and Rate Limit Gotchas

### Input + Output Tokens Both Count Toward Rate Limits
- Rate limits are based on **estimated max processed tokens** at request time, not actual tokens generated
- The estimate includes: prompt token count + `max_tokens` parameter + `best_of` parameter
- This means you can hit rate limits even when your actual token usage appears below quota in monitoring metrics
- **Tip**: Set `max_tokens` to the minimum value that serves your use case — don't leave it at the default maximum

### Rate Limits Are Per-Deployment, Not Per-Resource
- Each model deployment within a resource has its own TPM (Tokens-Per-Minute) and RPM (Requests-Per-Minute) limits
- Quota is allocated at the subscription level and distributed across deployments
- You can shift quota between deployments to match traffic patterns
- RPM is enforced on 1-second or 10-second windows, not smoothly over a minute — bursts trigger 429s even if average RPM is within limits

### 429 Errors When Metrics Show Headroom
- Token usage metrics reflect **billed tokens that were successfully processed**
- Rate limiting applies to **all API requests**, including ones that fail (HTTP 400) or are rejected
- Requests rejected for input length limits (400) still count toward rate limiting but don't appear in token metrics
- **Rely on**: HTTP 429 response codes for rate-limit detection, not Azure Monitor token metrics

## PTU (Provisioned Throughput) Gotchas

### PTU Doesn't Eliminate Throttling
- PTU provides **predictable capacity**, not unlimited capacity
- Exceeding your PTU allocation still results in 429 responses
- Output tokens consume more capacity than input tokens — the ratio varies by model (e.g., 4:1 for gpt-4.1)
- **Key difference**: PTU gives you a guaranteed floor of capacity with consistent latency; Standard gives you best-effort with variable latency

### PTU Billing Is Continuous
- PTU charges accrue whether you use the capacity or not — it's reserved compute
- Minimum deployments vary by model (e.g., 15 PTUs for GPT-4o global, 50 PTUs for GPT-4o regional)
- You cannot scale PTU deployments to zero — minimum deployment sizes apply

## Content Filtering Gotchas

### Filters Can Silently Modify or Block Responses
- Content filtering is applied by default to all Azure OpenAI deployments
- Filters evaluate both prompts (input) and completions (output) against four categories: hate, violence, sexual, self-harm
- A filtered completion returns HTTP 200 with `finish_reason: content_filter` — your application must check for this
- You are **billed** for requests that return 200 with `content_filter` finish reason — the model processed the tokens
- Custom filter configurations can adjust thresholds but cannot fully disable filtering

## Model Gotchas

### Model Deprecation Happens with Limited Notice
- Azure retires model versions on a published schedule, but the timeline can be tight
- Deprecated models stop accepting new deployments first, then existing deployments are retired
- **Action**: Subscribe to Azure OpenAI model deprecation notifications. Maintain a model upgrade path in your deployment automation. Test new model versions in staging regularly

### Response Streaming Behaves Differently Across Models
- Streaming response behavior (chunk size, timing, final chunk format) varies between model families
- Reasoning models (o1, o3, o4-mini) have different streaming patterns than chat models (GPT-4o, GPT-4.1)
- Some models include reasoning tokens that are billed but not visible in streamed output
- **Tip**: Test streaming behavior specifically for each model you deploy

### JSON Mode Edge Cases
- `response_format: { type: "json_object" }` encourages but does not guarantee valid JSON in all cases
- The model can still produce truncated JSON if it hits the `max_tokens` limit mid-object
- JSON mode requires mentioning "JSON" in the system or user message — omitting this can cause errors
- For strict schemas, use Structured Outputs (`response_format: { type: "json_schema" }`) where available

## Azure AI Search Gotchas

### Semantic Ranker Has Separate Per-Query Costs
- Semantic ranking (the neural re-ranker layer) is billed per query, separate from index storage and operations
- This cost is in addition to base search query costs and can add up quickly at scale
- The ranker processes the top 50 results from the initial retrieval — increasing `top` beyond 50 doesn't improve ranking
- **Tip**: Use semantic ranking selectively (e.g., user-facing queries) rather than on every internal retrieval call

### Embedding Dimensions Must Match Exactly
- Different embedding models produce different vector sizes (1536 for text-embedding-3-small, 3072 for text-embedding-3-large)
- An index configured for 1536-dimension vectors cannot accept 3072-dimension vectors — there's no automatic conversion
- Changing embedding models requires re-indexing all documents
- `text-embedding-3-*` models support dimension reduction via the `dimensions` parameter, but you must use the same setting at index and query time

## Authentication Gotchas

### Managed Identity Requires Specific RBAC Roles
- `Cognitive Services OpenAI User` — required for inference (chat, completions, embeddings)
- `Cognitive Services OpenAI Contributor` — required for managing deployments
- `Cognitive Services User` — grants access to all Cognitive Services, not just OpenAI (broader than needed)
- RBAC role assignments can take up to 10 minutes to propagate — new deployments may see 401 errors initially
- Using managed identity with Azure AI Search requires separate RBAC assignments on the search resource

## Fine-Tuning Gotchas

### Data Requirements Are Strict
- Training data must be in JSONL format with specific message structure (system/user/assistant)
- Minimum 10 examples required, but 50–100+ recommended for meaningful improvement
- Total training tokens have per-model limits — large datasets may need to be trimmed
- Fine-tuned models are tied to the region where they were trained — they cannot be moved
- Fine-tuning costs include both training hours and inference (which is more expensive than base model inference)
