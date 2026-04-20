# Model Selection Guide

## Flagship Models

### GPT-4o
- **Type**: General-purpose multimodal (text + image input, text output)
- **Strengths**: Excellent all-around performance, fast inference, cost-effective for its capability tier
- **Context window**: 128K tokens
- **Best for**: General chat, content generation, multimodal analysis, default choice when unsure
- **Cost tier**: Mid-range

### GPT-4.1
- **Type**: Latest flagship model, text-focused with multimodal support
- **Strengths**: Best-in-class for coding and instruction following, exceptional at long-context tasks
- **Context window**: 1M tokens (1,048,576)
- **Best for**: Complex coding tasks, precise instruction following, long document analysis, agentic workflows
- **Cost tier**: Premium
- **Note**: Supersedes GPT-4o for most tasks requiring highest quality

### GPT-4.1-mini
- **Type**: Smaller, optimized version of GPT-4.1
- **Strengths**: Excellent quality-to-cost ratio, fast, strong at coding and instruction following
- **Context window**: 1M tokens
- **Best for**: Most production workloads where GPT-4.1 quality isn't strictly necessary, high-volume applications
- **Cost tier**: Low (10-20x cheaper than flagship)
- **Note**: Supersedes GPT-4o-mini for most use cases

### GPT-4.1-nano
- **Type**: Smallest, fastest, cheapest model in the 4.1 family
- **Strengths**: Extremely fast, very low cost, good for simple structured tasks
- **Context window**: 1M tokens
- **Best for**: Classification, entity extraction, simple text generation, routing/triage, high-volume low-complexity tasks
- **Cost tier**: Minimal

## Reasoning Models

### o3
- **Type**: Deep reasoning model with extended thinking
- **Strengths**: Multi-step reasoning, complex math, advanced code generation, scientific analysis
- **Context window**: 200K tokens
- **Best for**: Complex problems requiring chain-of-thought, mathematical proofs, advanced code architecture
- **Cost tier**: Premium (reasoning tokens add cost)
- **Note**: Use only when standard models fail — reasoning tokens significantly increase cost

### o4-mini
- **Type**: Smaller reasoning model
- **Strengths**: Good reasoning capability at lower cost than o3, faster inference
- **Context window**: 200K tokens
- **Best for**: Moderate reasoning tasks, code debugging, structured analysis where o3 is overkill
- **Cost tier**: Mid-range for reasoning models

## Older Models (Still Available)

### GPT-4o-mini
- **Type**: Previous-generation small model
- **Status**: Being superseded by GPT-4.1-mini
- **Use case**: Existing deployments, backward compatibility
- **Recommendation**: Migrate to GPT-4.1-mini for new projects

## Embedding Models

### text-embedding-3-small
- **Dimensions**: 1536 (default), configurable down to 256
- **Best for**: Cost-effective embeddings, most RAG applications, semantic search
- **Cost**: ~$0.02 per 1M tokens
- **Recommendation**: Default choice for embeddings

### text-embedding-3-large
- **Dimensions**: 3072 (default), configurable
- **Best for**: Maximum retrieval quality, when small model quality is insufficient
- **Cost**: ~$0.13 per 1M tokens
- **Recommendation**: Use only when benchmarks show meaningful improvement over small

## Specialized Models

### DALL-E 3
- **Type**: Image generation from text prompts
- **Best for**: Marketing visuals, concept art, illustrations
- **Pricing**: Per-image (varies by resolution)

### Whisper
- **Type**: Speech-to-text transcription
- **Best for**: Audio transcription, meeting notes, subtitle generation
- **Supported**: Multiple languages, automatic language detection

## Model Selection Decision Tree

```
What's your task?
├─ Text generation, chat, general purpose?
│  ├─ Need absolute best quality? → GPT-4.1
│  ├─ Need good quality at lower cost? → GPT-4.1-mini
│  ├─ Need multimodal (image understanding)? → GPT-4o or GPT-4.1
│  └─ Simple classification/extraction? → GPT-4.1-nano
├─ Complex reasoning or multi-step problem solving?
│  ├─ Hard math, science, or architecture decisions? → o3
│  └─ Moderate reasoning, code debugging? → o4-mini
├─ Embeddings for search/RAG?
│  ├─ Default choice? → text-embedding-3-small
│  └─ Need maximum quality? → text-embedding-3-large
├─ Image generation? → DALL-E 3
└─ Speech-to-text? → Whisper
```

## Production Readiness Checklist

- **GA vs Preview**: Only GA models for production. Preview = potential breaking changes, no SLA.
- **Regional availability**: Not all models available in all regions. Check before committing to architecture.
- **Rate limits**: Default quota varies by model and subscription type. Request increases early.
- **SLA**: Provisioned = guaranteed throughput. Standard = best-effort. No SLA on free/developer tier.
- **Content filtering**: Verify filter configuration support for your chosen model.
- **Version pinning**: Pin to specific model versions in production. Test before upgrading.
- **Fine-tuning**: Available for GPT-4o-mini, GPT-4.1-mini. Not available for reasoning models.
