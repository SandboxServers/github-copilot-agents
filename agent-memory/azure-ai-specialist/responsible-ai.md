# Responsible AI Framework

## Azure OpenAI Content Filtering

Content filtering is applied by default to all Azure OpenAI deployments, covering both input (prompts) and output (completions).

### Filter Categories and Severity Levels

| Category | Applied To | Default Threshold | Severity Levels |
|----------|-----------|-------------------|-----------------|
| Hate and Fairness | Prompts + Completions | Medium | Safe, Low, Medium, High |
| Violence | Prompts + Completions | Medium | Safe, Low, Medium, High |
| Sexual | Prompts + Completions | Medium | Safe, Low, Medium, High |
| Self-Harm | Prompts + Completions | Medium | Safe, Low, Medium, High |

### Custom Content Filters
- Create custom filter configurations via Azure AI Foundry or REST API
- Adjust severity thresholds per category (e.g., allow Low violence for gaming content)
- Add custom blocklists for domain-specific terms or patterns
- Assign custom filters to specific deployments
- Content credentials (provenance tracking) for AI-generated images
- **Important**: Handle filter trigger responses gracefully in application code (HTTP 400 with specific error codes)

## Prompt Shields

### Jailbreak Detection
- Built-in classifier detects attempts to bypass system instructions
- Enabled by default on all deployments
- Catches common jailbreak patterns (role-play attacks, instruction override attempts)

### Indirect Prompt Injection Detection
- Detects malicious instructions embedded in retrieved content (RAG scenarios)
- Critical for RAG applications where user-supplied documents may contain adversarial content
- Scans retrieved context for injection attempts before sending to the model

## Groundedness Detection

- Checks whether the model's response is supported by the provided context (retrieved documents)
- Essential for RAG applications to prevent hallucination
- Available as a standalone API call or integrated into Azure AI Foundry evaluations
- Returns a groundedness score and identifies unsupported claims
- Use in production to flag or block ungrounded responses

## Protected Material Detection

### Text
- Detects known copyrighted text in model outputs (e.g., song lyrics, book excerpts)
- Enabled by default on completions

### Code
- Detects code from known open-source repositories in model outputs
- Identifies license type (MIT, GPL, Apache, etc.) when detected
- Enabled by default — helps manage open-source license compliance

## Azure AI Content Safety (Standalone Service)

Separate from Azure OpenAI content filtering — a standalone API for content moderation.

- **Text moderation**: Analyze text for harmful categories (same categories as OpenAI filters)
- **Image moderation**: Analyze images for harmful content
- **Multi-modal**: Combined text + image analysis
- **Custom categories**: Train custom classifiers for domain-specific harmful content
- **When to use**: Moderating user-generated content, non-LLM content pipelines, adding safety to applications that don't use Azure OpenAI

## Red Teaming

### Purpose
Systematically test AI applications for vulnerabilities, harmful outputs, and failure modes before deployment.

### Tools
- **PyRIT (Python Risk Identification Tool for GenAI)**: Microsoft's open-source red teaming framework
  - Automated adversarial prompt generation
  - Multiple attack strategies (jailbreak, prompt injection, harmful content elicitation)
  - Scoring and reporting
- **Azure AI Red Teaming Agent**: Managed red teaming within Azure AI Foundry

### Practice
- Red team before every production deployment and model update
- Test for domain-specific harms (not just generic categories)
- Include both automated testing (PyRIT) and manual testing by domain experts
- Align with MITRE ATLAS for AI threat modeling
- Reference OWASP Top 10 for LLM Applications
- Document findings and track remediation

## Transparency Best Practices

- **System messages**: Clearly state the AI's limitations, role, and boundaries
- **Disclosure**: Users should know they're interacting with AI
- **Citations**: RAG responses should cite source documents so users can verify claims
- **Confidence signals**: Surface uncertainty rather than presenting low-confidence answers as facts
- **Feedback mechanisms**: Allow users to report incorrect or harmful responses

## Data Privacy

### Azure OpenAI Data Handling
- **Customer data is NOT used for model training**: Azure OpenAI does not use prompts, completions, or fine-tuning data to train or improve base models
- **Data stays within Azure boundary**: Processed and stored within the Azure region/zone of your deployment
- **Abuse monitoring**: Microsoft may review flagged content for safety (opt-out available for approved customers)
- **No third-party access**: Your data is not shared with OpenAI or any third party
- **Retention**: Prompts and completions are stored temporarily for abuse monitoring (30 days by default), then deleted. Opt-out disables this.

### Compliance
- SOC 2 Type 2 certified
- HIPAA BAA available
- FedRAMP High (via Azure Government)
- GDPR compliant
- ISO 27001 certified

## Prompt Injection Defense Layers

A defense-in-depth approach for production AI applications:

1. **System prompt isolation**: Never expose system prompt content in user-facing output
2. **Input validation**: Content filters on user input (built-in by default)
3. **Jailbreak detection**: Built-in prompt shield classifier
4. **Indirect injection detection**: Scan retrieved content for adversarial instructions
5. **Output validation**: Content filters on model output (built-in by default)
6. **Groundedness checks**: Validate output against retrieved context in RAG scenarios
7. **Rate limiting**: Per-user and per-session limits to prevent automated abuse
8. **Monitoring and alerting**: Log filter triggers, jailbreak attempts, and anomalous patterns
