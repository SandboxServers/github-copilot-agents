# AI Service Selection Decision Tree

```
What's the AI use case?
├─ Pre-built capabilities (vision, speech, language, translation)?
│  └─ Azure AI Services (formerly Cognitive Services)
│     ├─ Vision: image analysis, OCR, Face, custom vision
│     ├─ Speech: speech-to-text, text-to-speech, translation, speaker recognition
│     ├─ Language: sentiment, NER, PII detection, summarization, QnA
│     └─ Translator: text translation, document translation
├─ Text generation, chat, reasoning, code?
│  ├─ Need Azure-hosted models with enterprise controls?
│  │  └─ Azure OpenAI Service (GPT-4o, GPT-4.1, o3, o4-mini)
│  ├─ Need open-source or third-party models?
│  │  └─ Azure AI Foundry Model Catalog
│  │     ├─ Models sold directly by Azure: DeepSeek, Llama, Grok (PTU eligible)
│  │     └─ Models sold by providers: Mistral, Cohere, etc (MaaS — pay-per-token)
│  └─ Need fine-tuned or custom models?
│     ├─ Fine-tuning available in Azure OpenAI (GPT-4o-mini, GPT-4.1-mini)
│     ├─ Fine-tuning in Foundry for open models
│     └─ Full custom training → Azure ML Compute (GPU clusters)
├─ RAG / knowledge grounding?
│  └─ See RAG Architecture section below
├─ AI agent / multi-step reasoning?
│  ├─ Microsoft Agent Framework (formerly AutoGen) — multi-agent orchestration
│  ├─ Semantic Kernel — SDK for AI app development (.NET, Python, Java)
│  └─ Azure AI Foundry Agent Service — managed agent hosting
└─ Batch processing / async?
   └─ Global Batch deployment type (50% discount, 24-hr SLA)
```
