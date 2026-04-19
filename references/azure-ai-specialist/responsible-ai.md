# Responsible AI Framework

## Content Filtering (Guardrails)

Default safety policies apply to all Azure OpenAI models:

| Category | Applied To | Default Threshold |
|----------|-----------|-------------------|
| Hate and Fairness | Prompts + Completions | Medium |
| Violence | Prompts + Completions | Medium |
| Sexual | Prompts + Completions | Medium |
| Self-Harm | Prompts + Completions | Medium |
| Prompt Injection (Jailbreak) | Prompts | On |
| Protected Material — Text | Completions | On |
| Protected Material — Code | Completions | On |

**Key points:**
- All filters are configurable (can increase/decrease severity thresholds)
- Custom blocklists for domain-specific terms
- Content credentials (provenance tracking) for AI-generated images
- Filters return specific error codes — handle them gracefully in application code

## Abuse Monitoring

Three components:
1. **Content Classification**: Real-time harmful content detection
2. **Abuse Pattern Capture**: Algorithmic detection of recurring harmful patterns (frequency, severity, intentionality)
3. **Review and Decision**: Automated (LLM-based) and human review for flagged content

**Opt-out**: Approved customers can request abuse monitoring exemption for sensitive use cases (e.g., healthcare). Must demonstrate alternative safety measures.

## Red Teaming Practice

- **Pre-deployment**: Manual red teaming to identify harm categories specific to your use case
- **Tools**: PyRIT (Python Risk Identification Tool for GenAI), Azure AI Red Teaming Agent
- **CI/CD integration**: Automated adversarial testing in deployment pipelines
- **Framework alignment**: MITRE ATLAS for AI threat modeling, OWASP Top 10 for LLM
- **Cadence**: Monthly or quarterly, plus on every model update or prompt change
- **Cross-functional**: AI devs + security + domain experts

## Prompt Injection Defense Layers

1. **System prompt isolation**: Never expose system prompt in user-facing output
2. **Input validation**: Content filter on user input (built-in)
3. **Jailbreak detection**: Built-in classifier (on by default)
4. **Output validation**: Content filter on model output
5. **Grounding checks**: Validate model output against retrieved data
6. **Rate limiting**: Per-user/per-session limits to prevent abuse
7. **Monitoring**: Log and alert on filter triggers, jailbreak attempts
