# Quick Start Guide

> Get started with OpenLLMetry in 5 minutes

---

## What is OpenLLMetry?

OpenLLMetry (Traceloop SDK) provides **automatic OpenTelemetry instrumentation** for LLM applications. Add one line of code and get complete observability for:

- 🤖 LLM calls (OpenAI, Anthropic, Bedrock, and 15+ more)
- 🔗 AI frameworks (LangChain, LlamaIndex, Haystack, etc.)
- 📊 Vector databases (Pinecone, ChromaDB, Qdrant, etc.)

No manual span creation needed - everything is automatic!

---

## Installation

```bash
pip install traceloop-sdk
```

That's it! The SDK automatically installs all necessary dependencies.

---

## Basic Usage

### Step 1: Initialize Traceloop

Add this at the **top** of your application (before importing LLM libraries):

```python
from traceloop.sdk import Traceloop

Traceloop.init(app_name="my-llm-app")
```

### Step 2: Use Your LLM Library Normally

```python
from openai import OpenAI

client = OpenAI()  # Automatically instrumented!

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "user", "content": "What is OpenTelemetry?"}
    ]
)

print(response.choices[0].message.content)
```

### Step 3: See Your Traces

By default, traces are printed to console. You'll see complete telemetry data including:

- ✅ Request parameters (model, temperature, etc.)
- ✅ Prompts and completions
- ✅ Token usage
- ✅ Response times
- ✅ Error details

---

## Complete Example

```python
from openai import OpenAI
from traceloop.sdk import Traceloop

# Initialize Traceloop
Traceloop.init(app_name="joke-generator")

# Use OpenAI as normal
client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a funny comedian."},
        {"role": "user", "content": "Tell me a joke about Python programming."}
    ],
    temperature=0.7
)

print(response.choices[0].message.content)

# Automatic trace data:
# - Model: gpt-4o-mini
# - Temperature: 0.7
# - Prompt tokens: ~25
# - Completion tokens: ~50
# - Duration: ~1.2s
# - Full request/response content
```

---

## Next Steps

### Send to Your Observability Backend

#### Option 1: Console Output (Development)

Default behavior - traces printed to stdout:

```python
Traceloop.init(app_name="my-app")
# Traces appear in console
```

#### Option 2: Jaeger (Local Development)

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

Traceloop.init(
    app_name="my-app",
    tracer_provider=provider
)
```

Run Jaeger locally:
```bash
docker run -d --name jaeger \
  -p 6831:6831/udp \
  -p 16686:16686 \
  jaegertracing/all-in-one:latest
```

View traces: http://localhost:16686

#### Option 3: OTLP Endpoint (Production)

```python
Traceloop.init(
    app_name="my-app",
    api_endpoint="https://your-otel-collector.com",
    headers={"Authorization": "Bearer your-token"}
)
```

Compatible with: Datadog, New Relic, Honeycomb, Grafana Cloud, AWS X-Ray, and any OTLP-compatible backend.

---

## Supported Libraries

### LLM Providers (17)

```python
# OpenAI
from openai import OpenAI
client = OpenAI()

# Azure OpenAI
from openai import AzureOpenAI
client = AzureOpenAI(...)

# Anthropic
import anthropic
client = anthropic.Anthropic()

# AWS Bedrock
import boto3
bedrock = boto3.client('bedrock-runtime')

# And 13 more: Cohere, Groq, Mistral, Ollama, Together,
# Replicate, Vertex AI, Google GenAI, Watsonx, etc.
```

**[See All Providers →](./instrumentation/llm-providers/README.md)**

### AI Frameworks (6)

```python
# LangChain
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_template("Tell me about {topic}")
model = ChatOpenAI()
chain = prompt | model

# LlamaIndex, Haystack, CrewAI, etc.
```

**[See All Frameworks →](./instrumentation/frameworks/README.md)**

### Vector Databases (7)

```python
# Pinecone
import pinecone
index = pinecone.Index("my-index")

# ChromaDB, Qdrant, Weaviate, Milvus, etc.
```

**[See All Vector DBs →](./instrumentation/vector-databases/README.md)**

---

## Configuration

### Control What's Captured

```python
import os

# Disable prompt/completion content for privacy
os.environ["TRACELOOP_TRACE_CONTENT"] = "false"

# Disable metrics
os.environ["TRACELOOP_METRICS_ENABLED"] = "false"

from traceloop.sdk import Traceloop
Traceloop.init(app_name="my-app")
```

### Add Custom Metadata

```python
from traceloop.sdk.tracing import set_association_properties

set_association_properties({
    "user_id": "user123",
    "session_id": "session456",
    "environment": "production"
})

# Now all traces include this metadata
```

### Custom Workflows

```python
from traceloop.sdk.decorators import workflow, task

@task(name="summarize_text")
def summarize(text):
    return llm.summarize(text)

@workflow(name="document_qa")
def qa_pipeline(document, question):
    summary = summarize(document)
    answer = llm.answer(summary, question)
    return answer

result = qa_pipeline(doc, "What is the main point?")
# Creates hierarchical trace:
# └─ document_qa.workflow
#    ├─ summarize_text.task
#    │  └─ OpenAI.chat
#    └─ answer_question.task
#       └─ OpenAI.chat
```

---

## Privacy & Security

### Content Capture Control

By default, OpenLLMetry captures prompts and completions for full visibility. For production or sensitive data:

```bash
export TRACELOOP_TRACE_CONTENT=false
```

**With content tracing disabled**:
- ✅ Model names, token counts, durations **still captured**
- ✅ Metrics **still collected**
- ❌ Actual text content **not captured**

### Best Practices

1. **Development**: Enable content tracing for debugging
2. **Staging**: Enable with sanitized data
3. **Production**: Disable or use with PII redaction

---

## Common Patterns

### Pattern 1: Simple LLM Call

```python
from openai import OpenAI
from traceloop.sdk import Traceloop

Traceloop.init(app_name="simple-app")
client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

### Pattern 2: Streaming

```python
stream = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Tell a story"}],
    stream=True
)

for chunk in stream:
    print(chunk.choices[0].delta.content or "", end="")

# Complete response automatically captured in trace
```

### Pattern 3: LangChain LCEL

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from traceloop.sdk import Traceloop

Traceloop.init(app_name="langchain-app")

prompt = ChatPromptTemplate.from_template("Joke about {topic}")
model = ChatOpenAI(model="gpt-4o-mini")
parser = StrOutputParser()

chain = prompt | model | parser
result = chain.invoke({"topic": "AI"})

# Automatic trace hierarchy:
# RunnableSequence.workflow
# ├─ ChatPromptTemplate.task
# ├─ ChatOpenAI.chat
# └─ StrOutputParser.task
```

### Pattern 4: LangChain Agent

```python
from langchain.agents import create_openai_tools_agent, AgentExecutor
from langchain.tools import tool
from traceloop.sdk import Traceloop

Traceloop.init(app_name="agent-app")

@tool
def search(query: str) -> str:
    """Search for information."""
    return "Search results..."

agent = create_openai_tools_agent(llm, tools=[search], prompt=prompt)
executor = AgentExecutor(agent=agent, tools=tools)

result = executor.invoke({"input": "Find info about OpenTelemetry"})

# Traces show:
# - Agent reasoning
# - Tool selections
# - Tool executions
# - Final synthesis
```

### Pattern 5: Anthropic Claude

```python
import anthropic
from traceloop.sdk import Traceloop

Traceloop.init(app_name="claude-app")
client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Explain quantum computing"}]
)

print(message.content[0].text)
```

### Pattern 6: AWS Bedrock

```python
import boto3
import json
from traceloop.sdk import Traceloop

Traceloop.init(app_name="bedrock-app")
bedrock = boto3.client('bedrock-runtime', region_name='us-east-1')

body = json.dumps({
    "anthropic_version": "bedrock-2023-05-31",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello!"}]
})

response = bedrock.invoke_model(
    modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
    body=body
)

result = json.loads(response['body'].read())
print(result['content'][0]['text'])
```

---

## Troubleshooting

### Issue: No traces appearing

**Solutions**:

1. **Verify initialization order** - Traceloop.init must come **before** library imports:

```python
# ✅ Correct
from traceloop.sdk import Traceloop
Traceloop.init(app_name="my-app")

from openai import OpenAI  # Import after init

# ❌ Wrong
from openai import OpenAI  # Import before init
from traceloop.sdk import Traceloop
Traceloop.init(app_name="my-app")
```

2. **Check exporter configuration**:

```python
from opentelemetry.sdk.trace.export import ConsoleSpanExporter
Traceloop.init(
    app_name="my-app",
    exporter=ConsoleSpanExporter()  # Forces console output
)
```

3. **Verify API key is set**:

```bash
export OPENAI_API_KEY=your-key-here
```

### Issue: Content not captured

**Solution**: Enable content tracing:

```bash
export TRACELOOP_TRACE_CONTENT=true
```

Or in Python:

```python
import os
os.environ["TRACELOOP_TRACE_CONTENT"] = "true"
```

### Issue: Import errors

**Solution**: Ensure library is installed:

```bash
pip install traceloop-sdk
pip install openai  # Or your LLM library
```

### Need More Help?

- **[Full SDK Guide](./traceloop-sdk-guide.md)** - Comprehensive documentation
- **[Provider-Specific Guides](./instrumentation/llm-providers/README.md)** - Detailed provider docs
- **[Examples](./examples-index.md)** - 64 working sample apps
- **[GitHub Issues](https://github.com/traceloop/openllmetry/issues)** - Report bugs or ask questions

---

## What's Next?

### Learn More

- **[Traceloop SDK Guide](./traceloop-sdk-guide.md)** - Deep dive into all features
- **[OpenAI Instrumentation](./instrumentation/llm-providers/openai.md)** - Azure, streaming, tools, vision
- **[Anthropic Instrumentation](./instrumentation/llm-providers/anthropic.md)** - Claude, caching, Bedrock
- **[LangChain Instrumentation](./instrumentation/frameworks/langchain.md)** - LCEL, agents, LangGraph
- **[AWS Bedrock Instrumentation](./instrumentation/llm-providers/bedrock.md)** - Multi-model, guardrails

### Explore Examples

Browse **[64 sample applications](./examples-index.md)** organized by:
- Provider (OpenAI, Anthropic, Bedrock, etc.)
- Framework (LangChain, LlamaIndex, etc.)
- Use case (streaming, agents, vision, etc.)
- Complexity (simple, intermediate, advanced)

### Join the Community

- **Slack**: [Join Traceloop Community](https://traceloop.com/slack)
- **GitHub**: [OpenLLMetry Repository](https://github.com/traceloop/openllmetry)
- **Discussions**: [GitHub Discussions](https://github.com/traceloop/openllmetry/discussions)

---

## Summary

**Three steps to full LLM observability**:

1. **Install**: `pip install traceloop-sdk`
2. **Initialize**: `Traceloop.init(app_name="my-app")`
3. **Run**: Use your LLM libraries normally - automatic tracing!

That's it! You now have complete visibility into:
- 🔍 Request/response data
- 💰 Token usage and costs
- ⏱️ Performance metrics
- 🐛 Error tracking
- 🔗 Multi-step workflows

**Happy tracing!** 🚀

---

**Last Updated**: 2025-11-22
