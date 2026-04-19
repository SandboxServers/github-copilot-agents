# Model Selection Guide

## Production Readiness Check

Before deploying any model from the Foundry catalog:
- **GA vs Preview**: Only GA models for production. Preview = breaking changes, no SLA.
- **Regional availability**: Check deployment regions. Not all models available everywhere.
- **Rate limits**: Default quota varies by model and subscription type. Request increases early.
- **SLA**: Provisioned = guaranteed throughput. Standard = best-effort. No SLA on Developer tier.
- **Content filtering**: Not all models support all filter configurations.
- **Fine-tuning availability**: Only select models support fine-tuning.

## Model Tier Selection

| Use Case | Recommended Model | Why |
|----------|------------------|-----|
| Complex reasoning, coding | GPT-4.1, o3 | Highest capability |
| General chat, content generation | GPT-4o | Good balance of quality/cost |
| Classification, extraction, simple tasks | GPT-4o-mini, GPT-4.1-mini | 10-20x cheaper, fast |
| Code generation | GPT-4.1, Claude Sonnet | Strong code capability |
| Embeddings | text-embedding-3-small | Cost-effective, good quality |
| Image generation | DALL-E 3, GPT-image-1 | Different style/control trade-offs |
| Open source alternative | DeepSeek-R1, Llama | PTU-eligible, no vendor lock |
