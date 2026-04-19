# Agent Framework & Orchestration

## Microsoft Agent Framework (formerly AutoGen/Magnetic-One)

- Multi-agent orchestration for complex AI workflows
- Supports Semantic Kernel VectorStore connectors for RAG
- `TextSearchProvider` for easy RAG integration
- Graph RAG support via Neo4j integration
- Managed hosting via Azure AI Foundry Agent Service

## Semantic Kernel

- SDK for building AI applications (.NET, Python, Java)
- Plugin system for extending AI capabilities
- VectorStore connectors for all major vector databases
- Function calling / tool use orchestration
- Process framework for multi-step workflows
- `create_search_function` bridges vector stores to agent tools

## When to Use What

| Need | Solution |
|------|----------|
| Simple chat completion | Direct Azure OpenAI API call |
| Chat with tools/functions | Semantic Kernel or Azure OpenAI function calling |
| RAG with search | AI Search + Semantic Kernel or direct API |
| Multi-agent orchestration | Microsoft Agent Framework |
| Visual AI workflow builder | Azure AI Foundry Prompt Flow |
| Managed agent hosting | Azure AI Foundry Agent Service |

## Prompt Flow Assessment

```
Is Prompt Flow the right choice?
├─ Need visual flow builder for non-developers? → Yes, Prompt Flow
├─ Need complex multi-step LLM chains? → Maybe — evaluate vs Semantic Kernel
├─ Need CI/CD integration and version control? → Prompt Flow supports it but code-first (SK) is easier
├─ Team already using Semantic Kernel? → Skip Prompt Flow, stay in code
├─ Need rapid prototyping before production? → Prompt Flow for prototype, SK for production
└─ Need maximum control and testability? → Semantic Kernel (code-first)
```
