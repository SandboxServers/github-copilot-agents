# Azure AI Anti-Patterns

Common mistakes that lead to security vulnerabilities, runaway costs, or unreliable AI applications.

## Security Anti-Patterns

### Hardcoding API Keys
- **Problem**: Embedding Azure OpenAI API keys in source code or config files checked into version control
- **Impact**: Key leakage exposes all deployments on that resource; no per-caller audit trail
- **Fix**: Use managed identity (system-assigned or user-assigned) with `Cognitive Services OpenAI User` RBAC role. Store any required secrets in Azure Key Vault, never in code

### Disabling Content Filtering
- **Problem**: Removing or weakening default content filters to avoid false positives
- **Impact**: Opens application to generating harmful content, violates Azure use policies, potential service suspension
- **Fix**: Keep filters enabled. Create custom filter configurations to tune severity thresholds per category. Handle filter trigger responses (HTTP 400) gracefully in application code

### Prompt Injection Vulnerabilities
- **Problem**: Passing unsanitized user input directly into prompts without separation or validation
- **Impact**: Users can override system instructions, extract system prompts, or cause the model to perform unintended actions
- **Fix**: Enable Prompt Shields (jailbreak + indirect injection detection). Separate system messages from user input. Validate and sanitize all user-supplied content before inclusion in prompts. Use Azure AI Content Safety for input scanning

## Cost Anti-Patterns

### No Token Budget Management
- **Problem**: Deploying AI endpoints without setting `max_tokens`, monitoring usage, or configuring cost alerts
- **Impact**: Runaway costs from unexpectedly long completions, loops, or abuse. A single misbehaving client can exhaust quota
- **Fix**: Set `max_tokens` to the minimum value that serves your use case. Implement per-user/per-session token budgets. Configure Azure Monitor alerts on `TokenTransaction` metrics. Use Azure Cost Management budgets with automated actions

### Using PTU Without Understanding the Economics
- **Problem**: Purchasing Provisioned Throughput Units assuming they eliminate all throttling
- **Impact**: PTU provides predictable capacity, not unlimited capacity. Exceeding PTU allocation still results in 429s
- **Fix**: Use the Azure AI Foundry PTU calculator to right-size. Monitor utilization metrics. Consider hybrid PTU + Standard deployments for burst traffic

## Reliability Anti-Patterns

### No Retry Logic for Transient Errors
- **Problem**: Treating 429 (rate limit) and 5xx errors as permanent failures
- **Impact**: Unnecessary request failures during transient capacity constraints; poor user experience
- **Fix**: Implement retry with exponential backoff and jitter. Respect `Retry-After` headers. Use circuit breakers for sustained failures to avoid cascading load

### Using Preview Models in Production Without Fallback
- **Problem**: Deploying preview or newly released model versions to production without a fallback deployment
- **Impact**: Preview models can be deprecated, have behavior changes, or experience availability issues without SLA coverage
- **Fix**: Always maintain a stable GA model deployment as fallback. Use API Management or application-level routing to switch models. Test new model versions in staging before promotion

### Not Implementing Circuit Breakers
- **Problem**: Continuing to send requests to a failing AI endpoint without backoff
- **Impact**: Cascading failures, exhausting client resources, amplifying load on already-stressed services
- **Fix**: Implement circuit breaker pattern — after N consecutive failures, stop sending requests for a cooldown period. Use Azure API Management with backend circuit breaker policies. Monitor with `AzureOpenAIAvailabilityRate` metric

## RAG Anti-Patterns

### Improper Document Chunking
- **Problem**: Using chunks that are too large (lose retrieval precision) or too small (lose context)
- **Impact**: Poor retrieval quality leads to irrelevant context, hallucinations, or incomplete answers
- **Fix**: Target 512–1024 tokens per chunk with 10–15% overlap. Preserve document structure (don't split mid-sentence or mid-table). Use semantic chunking where possible. Test with your actual queries

### Embedding Model Mismatch
- **Problem**: Using different embedding models for indexing documents and querying at search time
- **Impact**: Vector similarity breaks completely — different models produce different vector spaces and dimensions
- **Fix**: Use the same model and version for both indexing and querying. Record the model used in index metadata. When migrating models, re-index all documents

## Evaluation Anti-Patterns

### Ignoring Regional Availability
- **Problem**: Deploying to a single region without checking model availability or planning for regional failover
- **Impact**: Model availability varies by region. Service disruptions in one region take down entire application
- **Fix**: Check model availability per region before architecture decisions. Deploy to multiple regions for production workloads. Use Azure Traffic Manager or Front Door for failover

### No Evaluation or Testing of AI Outputs
- **Problem**: Deploying AI features without systematic testing of output quality, accuracy, or safety
- **Impact**: Hallucinations, incorrect answers, or harmful content reach end users undetected
- **Fix**: Implement automated evaluation using Azure AI Foundry evaluation tools. Build test datasets covering expected scenarios and edge cases. Use groundedness detection for RAG applications. Integrate PyRIT for adversarial testing in CI/CD

### Over-Relying on Temperature=0 for Determinism
- **Problem**: Assuming `temperature=0` guarantees identical outputs for identical inputs
- **Impact**: LLMs are inherently non-deterministic even at temperature=0 due to floating-point operations and infrastructure variations
- **Fix**: Use temperature=0 as a best-effort measure but design for variability. Implement output validation rather than expecting exact reproducibility. Use `seed` parameter where available for improved (not guaranteed) consistency
