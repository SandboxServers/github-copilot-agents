# Prompt Engineering Patterns & System Message Design

Practical patterns for designing effective prompts and system messages with Azure OpenAI.

## System Message Architecture

### Anatomy of an Effective System Message
Structure system messages in clear sections for maintainability:
```
1. Role/Persona definition — who the model is
2. Core instructions — what to do
3. Constraints — what NOT to do
4. Output format — how to structure responses
5. Examples — few-shot demonstrations
6. Safety/guardrails — boundaries and escalation rules
```

### Key Principles
- **Be explicit**: "You are a customer support agent for Contoso. You help with billing and account questions only." beats "You are helpful."
- **Define boundaries**: List topics the model should refuse or redirect — don't rely on implicit understanding
- **Repeat critical rules**: Place the most important constraints both at the beginning and end of the system message. Models weight recent tokens more heavily
- **Version your prompts**: Store system messages in source control. Track changes alongside model version changes

## Common Prompt Patterns

### Chain-of-Thought (CoT)
Force the model to show reasoning before answering:
```
Think through this step-by-step:
1. Identify the key entities in the question
2. Determine what information is needed
3. Reason about the answer
4. Provide your final answer
```
- Use for complex reasoning, math, multi-step analysis
- Increases token usage but improves accuracy significantly
- With reasoning models (o3, o4-mini), CoT is built in — don't add explicit CoT instructions

### Few-Shot Examples
Provide input/output pairs to establish patterns:
```
User: What's the refund policy for annual subscriptions?
Assistant: {"intent": "refund_policy", "product": "annual_subscription", "sentiment": "neutral"}

User: I'm furious that my order hasn't arrived after 3 weeks!
Assistant: {"intent": "order_status", "product": "unknown", "sentiment": "negative"}
```
- 2–5 examples is usually sufficient for format consistency
- Match examples to your actual distribution of inputs
- Include edge cases that demonstrate desired boundary behavior

### Retrieval-Augmented Prompt Structure
Standard pattern for RAG prompts:
```
## Instructions
Answer the user's question using ONLY the provided context.
If the context doesn't contain the answer, say "I don't have information about that."
Always cite the source document for each claim.

## Context
{retrieved_chunks}

## User Question
{user_query}
```
- Place context before the user question — models attend to nearby content more strongly
- Cap context to leave room for the response within the model's context window
- Include source metadata in context chunks for citation generation

### Structured Output Enforcement
Use JSON schema constraints for reliable structured output:
```python
response = client.chat.completions.create(
    model="gpt-4.1",
    messages=[...],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "analysis_result",
            "strict": True,
            "schema": { ... }
        }
    }
)
```
- Structured Outputs (`json_schema`) > JSON mode (`json_object`) > prompt-only JSON requests
- Structured Outputs guarantee schema compliance; JSON mode only guarantees valid JSON
- Always set `max_tokens` high enough to complete the JSON — truncated JSON is invalid JSON

## Anti-Patterns in Prompt Design

### Vague Instructions
- **Bad**: "Be helpful and answer questions"
- **Good**: "You are a technical support agent for Azure services. Answer questions about Azure OpenAI configuration and troubleshooting. If the question is about billing, direct the user to the Azure billing portal."

### Over-Stuffing Context
- Don't dump entire documents into the prompt — it dilutes relevance and wastes tokens
- Retrieve only the top-K most relevant chunks (typically K=3–5)
- For long documents, summarize or extract key sections before including as context

### Ignoring Token Limits
- System message + context + user message + expected completion must fit within the model's context window
- Reserve at least 25% of the context window for the completion
- Track total prompt size programmatically — don't discover truncation in production

## Temperature and Sampling Strategy

| Use Case | Temperature | Top-P | Notes |
|----------|------------|-------|-------|
| Classification / extraction | 0 | 1.0 | Maximum consistency |
| Customer support / Q&A | 0.3 | 0.95 | Slightly creative but grounded |
| Content generation | 0.7 | 0.95 | Creative but coherent |
| Brainstorming | 1.0 | 1.0 | Maximum diversity |

- Set **either** `temperature` or `top_p`, not both — they interact unpredictably
- `temperature=0` is not deterministic — results can still vary across requests
- Use the `seed` parameter for improved (not guaranteed) reproducibility
