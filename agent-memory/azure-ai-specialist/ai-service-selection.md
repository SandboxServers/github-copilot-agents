# AI Service Selection Guide

## Azure OpenAI Service

The primary service for generative AI workloads on Azure.

- **Models available**: GPT-4o, GPT-4.1, GPT-4.1-mini, GPT-4.1-nano, o3, o4-mini reasoning models, DALL-E 3, Whisper, embedding models
- **When to use**: Text generation, code generation, chat/conversational AI, embeddings for search/RAG, image generation, audio transcription
- **Requirements**: Azure subscription + access approval (approval is automatic for most models now)
- **Key features**: Enterprise security, content filtering, private endpoints, regional data residency, fine-tuning support
- **Deployment types**: Standard (pay-per-token), Provisioned (reserved capacity), Global, Data Zone, Batch

## Azure AI Foundry (formerly Azure AI Studio)

Unified portal and platform for building end-to-end AI applications.

- **Model catalog**: Access OpenAI models alongside Meta Llama, Mistral, Cohere, DeepSeek, and other open/third-party models
- **Prompt flow**: Visual orchestration for LLM chains and RAG pipelines
- **Evaluation tools**: Built-in metrics for groundedness, relevance, coherence, fluency
- **Agent Service**: Managed hosting for AI agents with built-in tool calling and code interpreter
- **When to use**: Building complete AI applications that need model experimentation, prompt engineering, evaluation, and deployment in one place
- **Relationship to Azure OpenAI**: Foundry is the portal layer; Azure OpenAI is the underlying model hosting service

## Azure AI Services (formerly Cognitive Services)

Pre-built AI models for specific perception and language tasks.

- **Vision**: Image analysis, OCR, Face detection, custom image classification
- **Speech**: Speech-to-text, text-to-speech, real-time translation, speaker recognition
- **Language**: Sentiment analysis, named entity recognition, PII detection, summarization, question answering, text analytics
- **Document Intelligence**: Extract structured data from forms, invoices, receipts, IDs
- **Translator**: Real-time text translation, document translation across 100+ languages
- **When to use**: You need a specific perception/language capability with a pre-built model. No ML expertise required. Pay-per-call pricing.
- **Key advantage**: Production-ready with minimal setup. SOC 2, HIPAA, FedRAMP compliant out of the box.

## Azure Machine Learning

Full ML platform for custom model training and MLOps.

- **Capabilities**: Custom model training, automated ML, managed compute (CPU/GPU clusters), managed endpoints for model serving, ML pipelines, model registry
- **When to use**: You need custom models trained on your data, not off-the-shelf models. Fine-tuning needs beyond what Azure OpenAI provides. Classical ML workloads (regression, classification, time series).
- **Key features**: MLOps with CI/CD, experiment tracking, model versioning, responsible AI dashboard
- **Not for**: Simple use of pre-trained models (use Azure OpenAI or AI Services instead)

## Decision Matrix

| Service | Best For | ML Expertise Required | Cost Model | Customization Level |
|---------|----------|----------------------|------------|-------------------|
| Azure OpenAI | Text generation, chat, code, embeddings | None | Per-token or PTU | Prompt engineering, fine-tuning |
| AI Foundry | End-to-end AI app building, model comparison | Low-Medium | Varies by model | Full pipeline customization |
| AI Services | Vision, speech, language, document tasks | None | Per-API-call | Limited (custom models for some) |
| Azure ML | Custom model training, classical ML, MLOps | High | Compute-based | Full custom training |

## Quick Selection Guide

```
What do you need?
├─ Generate text, chat, code, or reason? → Azure OpenAI Service
├─ Extract data from images, speech, or documents? → Azure AI Services
├─ Compare models from multiple providers? → Azure AI Foundry Model Catalog
├─ Build a complete AI app with eval and monitoring? → Azure AI Foundry
├─ Train a custom model on your own data? → Azure Machine Learning
├─ Deploy an AI agent with tools? → Azure AI Foundry Agent Service
└─ Batch process large datasets with LLMs? → Azure OpenAI Global Batch
```

## Common Mistakes

- **Using Azure ML when Azure OpenAI suffices**: If a pre-trained model solves your problem, don't train a custom one
- **Ignoring AI Services**: Vision, Speech, and Document Intelligence are mature, cheap, and easy — don't rebuild them with GPT
- **Confusing Foundry with OpenAI**: Foundry is the platform/portal; OpenAI is the model hosting service. You use both together.
- **Skipping evaluation**: AI Foundry evaluation tools exist for a reason. Measure before shipping.
