# Azure AI Best Practices

## Authentication & Identity

### Use Managed Identity, Never API Keys
- Assign `Cognitive Services OpenAI User` RBAC role for inference workloads
- Prefer user-assigned managed identity for production — survives resource recreation during deployments
- API keys are acceptable only for local development and quick prototyping
- If keys are unavoidable, store in Azure Key Vault with access policies, never in code or config

### Network Security
- Deploy Azure OpenAI behind private endpoints for production workloads
- Use Azure API Management as a gateway for centralized auth, rate limiting, and logging
- Configure NSG and firewall rules to restrict access to known client IP ranges or VNets

## Resilience & Error Handling

### Retry with Exponential Backoff
- Implement retries for 429 (rate limit), 500 (server error), and 503 (service unavailable)
- Use exponential backoff with jitter to avoid thundering herd on recovery
- Respect `Retry-After` headers — Azure OpenAI includes them on 429 responses
- Set a maximum retry count (3–5) to avoid infinite loops
- Use circuit breaker pattern after sustained failures — stop sending requests for a cooldown period

### Multi-Region Deployment
- Deploy to at least two regions for production workloads — model availability varies by region
- Use Azure API Management or Azure Front Door for automatic failover routing
- Monitor per-region availability with `AzureOpenAIAvailabilityRate` metric
- Keep deployment configurations consistent across regions (same model version, same content filter settings)

## Content Filtering & Responsible AI

### Enable and Monitor, Don't Disable
- Keep default content filters enabled — they cover hate, violence, sexual, and self-harm categories
- Create custom filter configurations to tune severity thresholds for your domain (e.g., medical, gaming)
- Monitor for `finish_reason: content_filter` in responses — track frequency to detect issues
- Enable Prompt Shields for jailbreak and indirect injection detection, especially in RAG applications
- Implement groundedness detection to catch hallucinated responses

## Token Management & Cost Optimization

### Track Usage and Set Budgets
- Set `max_tokens` to the minimum value that serves your use case — never leave at default maximum
- Monitor `ProcessedPromptTokens` and `GeneratedTokens` metrics per deployment in Azure Monitor
- Configure Azure Cost Management budgets with automated alerts at 50%, 80%, 100% thresholds
- Use `usage` field in API responses to track actual token consumption per request
- Implement per-user or per-session token budgets in application code

### Cache and Batch
- Cache frequent identical queries at the application layer — LLM responses for static prompts don't change meaningfully
- Batch embedding requests instead of sending one document at a time — reduces API call overhead
- Use Azure OpenAI Batch API for non-time-sensitive workloads (50% cost reduction)
- Consider prompt caching for long system messages that repeat across requests

## RAG Best Practices

### Optimal Chunking Strategy
- Target 512–1024 tokens per chunk as a starting point
- Use 10–15% overlap between chunks to preserve context at boundaries
- Preserve document structure — don't split mid-sentence, mid-paragraph, or mid-table
- Use semantic chunking (split on topic boundaries) where document structure allows
- Store chunk metadata (source document, page number, section heading) for citation

### Hybrid Search Configuration
- Always use hybrid search (keyword BM25 + vector + semantic ranker) as the default
- Pure vector search misses exact-match queries (IDs, codes, acronyms)
- Pure keyword search misses semantic meaning when users use different terminology
- Configure filterable metadata fields (source, date, category) for scoped retrieval
- Test retrieval quality with representative query sets before connecting to the LLM

## Prompt Engineering

### System Message Design
- Place core instructions, persona, and constraints in the system message
- Be explicit about what the model should and should not do
- Use structured formats: define output schema, provide examples of expected responses
- Repeat critical instructions at the end of long system messages — models weight recent tokens more heavily

### Few-Shot Examples
- Include 2–5 high-quality examples of input/output pairs for consistent formatting
- Match examples to your actual use cases — generic examples produce generic results
- For structured output, include examples of the exact JSON/format you expect

### Structured Output
- Use `response_format: { type: "json_schema" }` (Structured Outputs) for reliable JSON generation
- Fall back to `response_format: { type: "json_object" }` if Structured Outputs isn't available for your model
- Always mention "JSON" in the prompt when using JSON mode — it's required by the API

## Multi-Model Strategy

### Right Model for Right Task
- Use GPT-4.1/GPT-4o for complex reasoning, analysis, and generation tasks
- Use GPT-4.1-mini/GPT-4o-mini for high-volume, lower-complexity tasks (classification, extraction, summarization)
- Use o3/o4-mini for tasks requiring deep reasoning or multi-step problem solving
- Use text-embedding-3-small for most embedding use cases; text-embedding-3-large for maximum accuracy
- Benchmark models against your specific workload — don't assume newest = best for your task

## Monitoring & Observability

### Log Prompts and Completions (with PII Scrubbing)
- Log prompt templates and completions for debugging and quality analysis
- Scrub PII before logging — use Azure AI Content Safety or custom regex patterns
- Track key metrics: latency (Time to First Byte, Time Between Tokens), token usage, error rates, content filter triggers
- Set up Azure Monitor alerts on availability rate, latency P95, and 429 error spikes
- Use Application Insights for end-to-end request tracing through your AI pipeline

## Evaluation & Testing

### Automated Quality Testing
- Build evaluation datasets covering expected scenarios, edge cases, and adversarial inputs
- Use Azure AI Foundry evaluation tools for automated groundedness, relevance, and coherence scoring
- Integrate PyRIT (Python Risk Identification Tool) for adversarial/red-team testing in CI/CD
- Run evaluations on every model version change and major prompt update
- Track evaluation metrics over time to detect quality regressions
