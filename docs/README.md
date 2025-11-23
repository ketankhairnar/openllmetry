# OpenLLMetry Documentation

> **Comprehensive documentation for OpenLLMetry (Traceloop SDK)**
> OpenTelemetry-native LLM observability for Python applications

---

## 🚀 Quick Links

- **[Quick Start Guide](./quick-start.md)** - Get started in 5 minutes
- **[Traceloop SDK Guide](./traceloop-sdk-guide.md)** - Complete SDK reference
- **[Examples Index](./examples-index.md)** - 64 working sample applications

---

## 📚 Documentation Structure

### Core Documentation

| Document | Description | Status |
|----------|-------------|--------|
| **[Quick Start](./quick-start.md)** | 5-minute getting started guide | ✅ Complete |
| **[Traceloop SDK Guide](./traceloop-sdk-guide.md)** | Comprehensive SDK documentation | ✅ Complete |
| **[Examples Index](./examples-index.md)** | Catalog of all sample applications | ✅ Complete |

### Instrumentation Guides

#### LLM Providers

| Provider | Documentation | Status |
|----------|---------------|--------|
| **[OpenAI](./instrumentation/llm-providers/openai.md)** | OpenAI & Azure OpenAI | ✅ Complete |
| **[Anthropic](./instrumentation/llm-providers/anthropic.md)** | Claude models, Bedrock integration | ✅ Complete |
| **[AWS Bedrock](./instrumentation/llm-providers/bedrock.md)** | Multi-model, guardrails, caching | ✅ Complete |
| Cohere | Command models | 📋 Planned |
| Groq | Fast inference | 📋 Planned |
| Mistral AI | Mistral models | 📋 Planned |
| Ollama | Local models | 📋 Planned |
| Vertex AI | Google Cloud AI | 📋 Planned |
| Others | 10+ additional providers | 📋 Planned |

**[Browse All LLM Providers →](./instrumentation/llm-providers/README.md)**

#### Frameworks

| Framework | Documentation | Status |
|-----------|---------------|--------|
| **[LangChain](./instrumentation/frameworks/langchain.md)** | LCEL, agents, tools, LangGraph | ✅ Complete |
| LlamaIndex | Indexes, query engines | 📋 Planned |
| Haystack | Pipelines, agents | 📋 Planned |
| CrewAI | Multi-agent systems | 📋 Planned |

**[Browse All Frameworks →](./instrumentation/frameworks/README.md)**

#### Vector Databases

| Database | Documentation | Status |
|----------|---------------|--------|
| Pinecone | Vector operations | 📋 Planned |
| ChromaDB | Local/cloud | 📋 Planned |
| Qdrant | High-performance | 📋 Planned |
| Weaviate | GraphQL API | 📋 Planned |
| Others | Milvus, LanceDB, Marqo | 📋 Planned |

**[Browse All Vector Databases →](./instrumentation/vector-databases/README.md)**

---

## 🎯 Getting Started

### Installation

```bash
pip install traceloop-sdk
```

### Basic Usage

```python
from openai import OpenAI
from traceloop.sdk import Traceloop

# Initialize Traceloop - automatically instruments supported libraries
Traceloop.init(app_name="my-llm-app")

# Use any supported library - automatically traced
client = OpenAI()
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello!"}]
)

print(response.choices[0].message.content)
```

**That's it!** Your LLM calls are now traced with:
- ✅ Complete request/response data
- ✅ Token usage and costs
- ✅ Performance metrics
- ✅ Error tracking

### What Gets Instrumented

**Automatically supported out of the box**:

**LLM Providers** (17 packages):
- OpenAI, Azure OpenAI, Anthropic Claude, AWS Bedrock
- Cohere, Groq, Mistral AI, Ollama, Together, Replicate
- Vertex AI, Google GenAI, IBM Watsonx, HuggingFace
- Aleph Alpha, Writer, SageMaker, OpenAI Agents

**Frameworks** (6 packages):
- LangChain (LCEL, agents, tools, LangGraph)
- LlamaIndex, Haystack, CrewAI, AutoGen, MCP

**Vector Databases** (7 packages):
- Pinecone, ChromaDB, Qdrant, Weaviate, Milvus, LanceDB, Marqo

**Total**: 30 instrumentation packages

---

## 📖 Documentation by Use Case

### I Want To...

#### Instrument OpenAI Calls
→ **[OpenAI Instrumentation Guide](./instrumentation/llm-providers/openai.md)**
- Chat completions, embeddings, assistants
- Streaming, async, vision, tools
- Azure OpenAI integration
- 10 working examples

#### Use Claude (Anthropic)
→ **[Anthropic Instrumentation Guide](./instrumentation/llm-providers/anthropic.md)**
- Messages API, extended thinking
- Prompt caching, tool use
- AWS Bedrock integration
- 5 working examples

#### Use AWS Bedrock
→ **[Bedrock Instrumentation Guide](./instrumentation/llm-providers/bedrock.md)**
- Multi-model support (6 providers)
- Guardrails with detailed metrics
- Prompt caching, cross-region inference
- Converse API

#### Build with LangChain
→ **[LangChain Instrumentation Guide](./instrumentation/frameworks/langchain.md)**
- LCEL pipelines, agents, tools
- LangGraph workflows
- 12+ LLM provider support
- 5 working examples

#### Understand the SDK
→ **[Traceloop SDK Guide](./traceloop-sdk-guide.md)**
- Installation and initialization
- Auto vs manual instrumentation
- Workflow and task decorators
- Configuration options
- Best practices

#### Browse Examples
→ **[Examples Index](./examples-index.md)**
- 64 sample applications
- Organized by provider, framework, use case
- Code snippets and file paths
- Complexity ratings

---

## 🏗️ Architecture

### How It Works

```
Your App
    ↓
Traceloop SDK (auto-instrumentation)
    ↓
Instrumentation Packages (OpenAI, Anthropic, LangChain, etc.)
    ↓
OpenTelemetry (standard observability protocol)
    ↓
Your Backend (Jaeger, Prometheus, Datadog, etc.)
```

### Key Components

1. **Traceloop SDK** - Main entry point, initializes instrumentation
2. **Instrumentation Packages** - Provider/framework-specific tracing
3. **OpenTelemetry** - Standard telemetry protocol
4. **Exporters** - Send data to your observability backend

### Data Collected

**Traces (Spans)**:
- Request parameters (model, temperature, max_tokens, etc.)
- Prompts and completions (configurable)
- Token usage and costs
- Performance metrics
- Error details

**Metrics**:
- Token usage histograms
- Request duration histograms
- Choice counters
- Exception counters
- Provider-specific metrics (guardrails, caching, etc.)

**Events** (optional):
- Input/output messages
- Choice events
- Custom application events

---

## ⚙️ Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `TRACELOOP_TRACE_CONTENT` | `"true"` | Capture prompts and completions |
| `TRACELOOP_METRICS_ENABLED` | `"true"` | Enable metrics collection |

### Privacy Control

```python
# Disable content capture for privacy
import os
os.environ["TRACELOOP_TRACE_CONTENT"] = "false"

# Still captures:
# ✅ Model names, token counts, durations
# ✅ Metrics and performance data
# ❌ Actual prompt/completion text
```

### Custom Configuration

```python
from traceloop.sdk import Traceloop

Traceloop.init(
    app_name="my-app",
    api_endpoint="https://my-otel-collector.com",
    headers={"Authorization": "Bearer token"},
    disable_batch=False,
    resource_attributes={
        "service.version": "1.0.0",
        "deployment.environment": "production"
    }
)
```

**[See Full SDK Configuration →](./traceloop-sdk-guide.md#configuration-options)**

---

## 🎓 Concepts

### Workflows and Tasks

OpenLLMetry uses hierarchical tracing:

- **Workflows** - Top-level operations (e.g., "customer support flow")
- **Tasks** - Sub-operations (e.g., "generate summary", "classify intent")
- **Tools** - Tool executions (e.g., "search database", "call API")

```python
from traceloop.sdk.decorators import workflow, task

@task(name="summarize")
def summarize_text(text):
    return llm.summarize(text)

@workflow(name="support_ticket")
def handle_ticket(ticket):
    summary = summarize_text(ticket.description)
    response = generate_response(summary)
    return response
```

### Semantic Conventions

OpenLLMetry follows OpenTelemetry GenAI semantic conventions:

- Standardized attribute names (`gen_ai.*`, `llm.*`)
- Consistent span structures
- Interoperable with OTel ecosystem
- Future-proof as standards evolve

**[Learn More →](https://opentelemetry.io/docs/specs/semconv/gen-ai/)**

### Association Properties

Add custom metadata to traces:

```python
from traceloop.sdk.tracing import set_association_properties

set_association_properties({
    "user_id": "user123",
    "session_id": "session456",
    "environment": "production"
})

# Appears in all spans as:
# traceloop.association_properties.user_id = "user123"
```

---

## 🔍 Advanced Topics

### Streaming Support

All instrumentation packages support streaming:

```python
# OpenAI streaming
stream = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Tell a story"}],
    stream=True
)

for chunk in stream:
    print(chunk.choices[0].delta.content, end="")

# Complete response captured in span
# Token usage tracked accurately
```

### Async Operations

Full async support across all packages:

```python
import asyncio
from openai import AsyncOpenAI

client = AsyncOpenAI()

async def generate():
    response = await client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": "Hello!"}]
    )
    return response.choices[0].message.content

asyncio.run(generate())
```

### Tool Calling

Tool/function calling automatically instrumented:

```python
tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get current weather",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {"type": "string"}
            }
        }
    }
}]

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Weather in SF?"}],
    tools=tools
)

# Tool definitions and calls captured in span
```

### Prompt Caching

Automatic detection and tracking:

```python
# Anthropic prompt caching
message = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    system=[{
        "type": "text",
        "text": "Long system prompt...",
        "cache_control": {"type": "ephemeral"}
    }],
    messages=[{"role": "user", "content": "Query"}]
)

# Cache metrics captured:
# - gen_ai.usage.cache_read_input_tokens
# - gen_ai.usage.cache_creation_input_tokens
```

---

## 📊 Integration Examples

### Jaeger (Local Development)

```python
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter

provider = TracerProvider()
jaeger_exporter = JaegerExporter(
    agent_host_name="localhost",
    agent_port=6831,
)
provider.add_span_processor(BatchSpanProcessor(jaeger_exporter))

Traceloop.init(tracer_provider=provider)
```

### Datadog

```python
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

Traceloop.init(
    api_endpoint="https://api.datadoghq.com",
    headers={"DD-API-KEY": "your-api-key"}
)
```

### Console (Debugging)

```python
from opentelemetry.sdk.trace.export import ConsoleSpanExporter

Traceloop.init(exporter=ConsoleSpanExporter())
```

**[See More Integrations →](./traceloop-sdk-guide.md#exporter-configuration)**

---

## 🛠️ Troubleshooting

### Common Issues

**No traces appearing?**
→ Verify initialization order (Traceloop.init before imports)
→ Check exporter configuration
→ See [Troubleshooting Guide](./traceloop-sdk-guide.md#troubleshooting)

**Content not captured?**
→ Ensure `TRACELOOP_TRACE_CONTENT=true`
→ Check privacy settings

**High cardinality metrics?**
→ Filter association properties
→ Avoid user IDs in metadata

**Provider-specific issues?**
→ Check provider documentation:
- [OpenAI Troubleshooting](./instrumentation/llm-providers/openai.md#troubleshooting)
- [Anthropic Troubleshooting](./instrumentation/llm-providers/anthropic.md#troubleshooting)
- [Bedrock Troubleshooting](./instrumentation/llm-providers/bedrock.md#troubleshooting)
- [LangChain Troubleshooting](./instrumentation/frameworks/langchain.md#troubleshooting)

---

## 📝 Best Practices

### 1. Use Descriptive Workflow Names

```python
# Good
@workflow(name="customer_support_ticket_handler")
def handle_support_ticket(ticket):
    ...

# Less helpful
@workflow(name="handler")
def handle_support_ticket(ticket):
    ...
```

### 2. Add Context via Association Properties

```python
set_association_properties({
    "user_id": user.id,
    "organization_id": org.id,
    "environment": "production"
})
```

### 3. Control Content Capture Appropriately

```python
# Production - disable for privacy
os.environ["TRACELOOP_TRACE_CONTENT"] = "false"

# Development - enable for debugging
os.environ["TRACELOOP_TRACE_CONTENT"] = "true"
```

### 4. Use Workflow Decorators for Logical Boundaries

```python
@workflow(name="document_qa_pipeline")
def qa_pipeline(document, question):
    chunks = split_document(document)
    relevant = find_relevant_chunks(chunks, question)
    answer = generate_answer(relevant, question)
    return answer
```

### 5. Monitor Costs via Token Metrics

```python
# Token histograms available in your observability backend
# - gen_ai.token.usage (by provider, model, type)
# - Calculate costs from token counts
```

---

## 🤝 Contributing

OpenLLMetry is open source and welcomes contributions!

- **Repository**: https://github.com/traceloop/openllmetry
- **Issues**: https://github.com/traceloop/openllmetry/issues
- **Discussions**: https://github.com/traceloop/openllmetry/discussions

---

## 📄 License

OpenLLMetry is licensed under the Apache License 2.0.

---

## 🔗 Additional Resources

- **OpenTelemetry**: https://opentelemetry.io
- **GenAI Semantic Conventions**: https://opentelemetry.io/docs/specs/semconv/gen-ai/
- **Traceloop Website**: https://www.traceloop.com
- **Community Slack**: [Join here](https://traceloop.com/slack)

---

**Last Updated**: 2025-11-22 | **Documentation Version**: 1.0
