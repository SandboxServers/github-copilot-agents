# Agent Frameworks & Orchestration

## Semantic Kernel (Microsoft's Recommended SDK)

Semantic Kernel is Microsoft's primary SDK for building AI-powered applications and agents.

### Overview
- **Languages**: .NET (C#), Python, Java
- **Purpose**: SDK for integrating LLMs into applications with plugins, planners, and memory
- **Maintained by**: Microsoft (first-party, deeply integrated with Azure OpenAI)

### Key Capabilities
- **Plugins**: Encapsulate functions (native code or LLM prompts) as reusable capabilities
- **Function calling**: Orchestrates tool use with Azure OpenAI function calling API
- **Planners**: AI-driven planning to decompose complex tasks into plugin calls
- **Memory**: Vector store connectors for RAG and conversation history
- **Process framework**: Multi-step workflow orchestration with state management
- **Filters**: Middleware pipeline for logging, content filtering, retry logic

### When to Use
- Enterprise .NET or Python applications integrating Azure OpenAI
- Production systems needing robust error handling, telemetry, and maintainability
- Teams already in the Microsoft ecosystem (Azure, .NET, Visual Studio)
- Applications requiring plugin-based extensibility

### Example Pattern (Python)
```python
import semantic_kernel as sk
from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion

kernel = sk.Kernel()
kernel.add_service(AzureChatCompletion(
    deployment_name="gpt-4.1",
    endpoint=endpoint,
    api_key=api_key  # or use DefaultAzureCredential
))
# Add plugins, invoke functions, etc.
```

## Azure AI Agent Service

Managed agent hosting within Azure AI Foundry.

### Overview
- **Type**: Fully managed agent runtime in Azure
- **Purpose**: Deploy agents without managing infrastructure
- **Integration**: Built into Azure AI Foundry portal

### Key Capabilities
- **Built-in tool calling**: Function calling orchestration managed by the service
- **Code interpreter**: Sandboxed Python execution for data analysis, chart generation
- **File search**: Built-in RAG over uploaded files (uses Azure AI Search under the hood)
- **Custom tools**: Define your own API-backed tools for the agent to call
- **Conversation management**: Thread and message management, persistent context

### When to Use
- Azure-native agent deployments where you want managed infrastructure
- Agents that need code execution (data analysis, file processing)
- Quick prototyping of agent capabilities before custom implementation
- Teams that prefer portal-based configuration over code-first

## AutoGen (Microsoft Research)

Multi-agent conversation framework for complex AI workflows.

### Overview
- **Type**: Open-source multi-agent framework
- **Languages**: Python (primary), .NET
- **Origin**: Microsoft Research, now open-source
- **Evolution**: AutoGen → Magnetic-One → Microsoft Agent Framework

### Key Capabilities
- **Multi-agent conversations**: Multiple agents with different roles collaborating
- **Customizable agents**: Define agent behaviors, tools, and interaction patterns
- **Group chat**: Orchestrate conversations between multiple agents
- **Human-in-the-loop**: Inject human feedback at any point in agent conversations
- **Code execution**: Agents can write and execute code in sandboxed environments

### When to Use
- Research and prototyping of multi-agent systems
- Complex workflows requiring multiple specialized agents
- Academic and experimental AI agent development
- Scenarios where agent-to-agent collaboration adds value

## LangChain

Popular open-source framework for LLM application development.

### Overview
- **Languages**: Python, JavaScript/TypeScript
- **Ecosystem**: Large community, extensive third-party integrations
- **Maintained by**: LangChain Inc. (open-source with commercial products)

### Key Capabilities
- **Chains**: Compose LLM calls with other operations in sequence
- **Agents**: ReAct-style agents with tool use
- **Retrievers**: Integrations with many vector stores and retrieval systems
- **LangSmith**: Observability and evaluation platform (commercial add-on)
- **LangGraph**: State machine framework for complex agent workflows

### When to Use
- Rapid prototyping when you need specific integrations not available in Semantic Kernel
- Python-first teams with existing LangChain experience
- Projects requiring integrations with non-Microsoft services (Pinecone, Weaviate, etc.)
- When LangGraph's state machine model fits your workflow better than Semantic Kernel's approach

## Framework Selection Guide

| Scenario | Recommended Framework | Reasoning |
|----------|----------------------|-----------|
| Enterprise .NET application | Semantic Kernel | First-party Microsoft, best .NET support |
| Enterprise Python application | Semantic Kernel | First-party, Azure-integrated |
| Azure-managed agent deployment | Azure AI Agent Service | Managed infrastructure, built-in tools |
| Multi-agent research/prototyping | AutoGen / Agent Framework | Purpose-built for multi-agent patterns |
| Rapid prototyping with many integrations | LangChain | Large ecosystem, fast iteration |
| Production with maximum Azure integration | Semantic Kernel + Agent Service | Deepest Azure OpenAI integration |
| Simple single-turn API calls | Direct Azure OpenAI SDK | No framework needed for simple calls |

## Tool Calling Patterns with Azure OpenAI

Azure OpenAI function calling is the foundation for agent tool use across all frameworks.

### How It Works
1. Define tools as JSON schemas (function name, description, parameters)
2. Send tools alongside the user message to the model
3. Model decides whether to call a tool and returns structured tool call(s)
4. Application executes the tool and returns results to the model
5. Model generates a final response incorporating tool results

### Best Practices
- **Descriptive function names and descriptions**: The model uses these to decide when to call tools
- **Strict schemas**: Use `strict: true` for guaranteed JSON conformance to your schema
- **Parallel tool calls**: Models can request multiple tool calls in a single turn — handle them concurrently
- **Error handling**: Return clear error messages from tools so the model can recover
- **Tool count**: Keep tools focused. 10-20 well-defined tools outperform 100 vague ones.

### Supported Models
- GPT-4o, GPT-4.1, GPT-4.1-mini, GPT-4.1-nano: Full function calling support
- o3, o4-mini: Function calling supported (reasoning models may be slower but more accurate at tool selection)

## Common Mistakes

- **Choosing LangChain for production Azure workloads**: Semantic Kernel has deeper Azure integration and Microsoft support
- **Over-engineering multi-agent**: Most real-world problems need one agent with good tools, not multiple agents talking to each other
- **Skipping direct API calls**: For simple prompt-in/response-out, you don't need a framework at all
- **Ignoring observability**: Whichever framework you choose, instrument it with tracing from day one
