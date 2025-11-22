# Traceloop SDK Guide

> **Status**: ✅ Complete (Continuously Improving)
> **Last Updated**: 2025-11-22
> **Package**: `traceloop-sdk`
> **Package Path**: `packages/traceloop-sdk/`

---

## Table of Contents

1. [Overview](#overview)
2. [Installation](#installation)
3. [Quick Start](#quick-start)
4. [Auto-Instrumentation](#auto-instrumentation)
5. [Configuration](#configuration)
6. [Manual Instrumentation](#manual-instrumentation)
7. [Advanced Features](#advanced-features)
8. [Best Practices](#best-practices)
9. [Troubleshooting](#troubleshooting)

---

## Overview

The **Traceloop SDK** is a comprehensive observability solution for LLM applications, built on [OpenTelemetry](https://opentelemetry.io/). It provides:

- ✅ **Zero-code auto-instrumentation** for 29+ AI libraries and frameworks
- ✅ **Manual instrumentation decorators** for custom workflows
- ✅ **Full OpenTelemetry compliance** - works with any OTLP backend
- ✅ **Semantic conventions** following [OpenTelemetry GenAI spec](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
- ✅ **Privacy controls** for sensitive data
- ✅ **Production-ready** with minimal performance overhead

### What Gets Auto-Instrumented?

**LLM Providers** (17):
- OpenAI, Anthropic, Cohere, Mistral, Ollama, Groq
- AWS Bedrock, SageMaker, Vertex AI, Google GenAI
- IBM Watsonx, Aleph Alpha, Writer, Together, Replicate
- HuggingFace Transformers, OpenAI Agents

**AI Frameworks** (6):
- LangChain (including LCEL and LangGraph)
- LlamaIndex, Haystack, CrewAI
- Model Context Protocol (MCP)

**Vector Databases** (7):
- Pinecone, Chroma, Qdrant, Weaviate
- Milvus, LanceDB, Marqo

**Infrastructure**:
- Redis, PyMySQL, Requests, urllib3

---

## Installation

### Basic Installation

```bash
pip install traceloop-sdk
```

### With Specific Instrumentations

The SDK will automatically instrument any installed libraries. Just install what you need:

```bash
# For OpenAI
pip install openai traceloop-sdk

# For LangChain
pip install langchain traceloop-sdk

# For Pinecone RAG
pip install openai pinecone-client chromadb traceloop-sdk
```

### Requirements

- Python 3.9+
- OpenTelemetry SDK dependencies (installed automatically)

---

## Quick Start

### 30-Second Setup

```python
from traceloop.sdk import Traceloop

# 1. Initialize once at app startup
Traceloop.init(app_name="my-llm-app")

# 2. Use your AI libraries normally - they're auto-instrumented!
from openai import OpenAI

client = OpenAI()
response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Hello!"}]
)

# ✅ Automatically traced:
#    - Span created: "openai.chat"
#    - Captured: model, prompts, completion, tokens
#    - Sent to your configured backend
```

**That's it!** No code changes needed for basic observability.

**Sample Reference**: [`packages/sample-app/sample_app/openai_streaming.py`](../packages/sample-app/sample_app/openai_streaming.py)

### Viewing Traces

#### Option 1: Console Output (Debugging)

```python
from opentelemetry.sdk.trace.export import ConsoleSpanExporter
from traceloop.sdk import Traceloop

Traceloop.init(
    app_name="debug-app",
    exporter=ConsoleSpanExporter()
)

# Traces printed to console in JSON format
```

#### Option 2: Traceloop Cloud (Managed)

```python
Traceloop.init(
    app_name="my-app",
    api_key="your-api-key"  # Or set TRACELOOP_API_KEY env var
)
```

#### Option 3: Custom OTLP Backend (Jaeger, Tempo, etc.)

```python
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter

Traceloop.init(
    app_name="my-app",
    exporter=OTLPSpanExporter(endpoint="http://localhost:4318/v1/traces")
)
```

---

## Auto-Instrumentation

### How It Works

When you call `Traceloop.init()`, the SDK:

1. **Creates a singleton `TracerProvider`** - The OpenTelemetry trace manager
2. **Detects installed libraries** - Checks for supported packages
3. **Injects instrumentors** - Automatically wraps library methods
4. **Configures exporters** - Sends traces to your backend

**All automatically** - no manual setup required.

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Your Application                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   OpenAI     │  │  LangChain   │  │  Pinecone    │     │
│  │   Calls      │  │   Chains     │  │   Queries    │     │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘     │
│         │                  │                  │              │
│         └──────────────────┼──────────────────┘              │
│                            │                                 │
│  ┌─────────────────────────▼──────────────────────────┐    │
│  │         Traceloop SDK (Instrumentors)              │    │
│  │  - Wraps library methods automatically             │    │
│  │  - Creates spans with semantic conventions         │    │
│  │  - Captures prompts, responses, tokens             │    │
│  └─────────────────────────┬──────────────────────────┘    │
│                            │                                 │
│  ┌─────────────────────────▼──────────────────────────┐    │
│  │       OpenTelemetry TracerProvider                 │    │
│  │  - Span processors (Batch/Simple)                  │    │
│  │  - Samplers (Always/Probabilistic)                 │    │
│  │  - Context propagation                             │    │
│  └─────────────────────────┬──────────────────────────┘    │
│                            │                                 │
│  ┌─────────────────────────▼──────────────────────────┐    │
│  │              Exporters                              │    │
│  │  - OTLP (HTTP/GRPC)                                │    │
│  │  - Console (debugging)                             │    │
│  │  - Custom exporters                                │    │
│  └─────────────────────────┬──────────────────────────┘    │
│                            │                                 │
└────────────────────────────┼─────────────────────────────────┘
                             │
                             ▼
              ┌─────────────────────────┐
              │  Observability Backend  │
              │  (Traceloop, Jaeger,    │
              │   Tempo, DataDog, etc.) │
              └─────────────────────────┘
```

### Selective Instrumentation

By default, **all** detected libraries are instrumented. To instrument only specific libraries:

```python
from traceloop.sdk import Traceloop
from traceloop.sdk.instruments import Instruments

# Instrument only OpenAI and Pinecone
Traceloop.init(
    app_name="my-app",
    instruments={Instruments.OPENAI, Instruments.PINECONE}
)
```

**Available Instruments**:
```python
# LLM Providers
Instruments.OPENAI
Instruments.ANTHROPIC
Instruments.COHERE
Instruments.MISTRAL
Instruments.OLLAMA
Instruments.GROQ
Instruments.BEDROCK
Instruments.SAGEMAKER
Instruments.VERTEXAI
Instruments.GOOGLE_GENERATIVEAI
Instruments.WATSONX
Instruments.ALEPHALPHA
Instruments.WRITER
Instruments.TOGETHER
Instruments.TRANSFORMERS
Instruments.REPLICATE
Instruments.OPENAI_AGENTS

# Vector DBs
Instruments.PINECONE
Instruments.QDRANT
Instruments.CHROMA
Instruments.WEAVIATE
Instruments.MILVUS
Instruments.MARQO
Instruments.LANCEDB

# Frameworks
Instruments.LANGCHAIN
Instruments.LLAMA_INDEX
Instruments.HAYSTACK
Instruments.CREWAI
Instruments.MCP

# Infrastructure
Instruments.REDIS
Instruments.PYMYSQL
Instruments.REQUESTS
Instruments.URLLIB3
```

### Blocking Specific Instrumentations

```python
# Instrument everything EXCEPT LangChain
Traceloop.init(
    app_name="my-app",
    block_instruments={Instruments.LANGCHAIN}
)
```

---

## Configuration

### Initialization Parameters

```python
Traceloop.init(
    # === Required ===
    app_name: str,                          # Application name (service.name)

    # === Traceloop Cloud (Optional) ===
    api_endpoint: str = None,               # Traceloop API endpoint
    api_key: str = None,                    # Traceloop API key (or TRACELOOP_API_KEY)

    # === Enable/Disable ===
    enabled: bool = True,                   # Master switch for tracing
    telemetry_enabled: bool = True,         # Anonymous usage telemetry

    # === Exporters ===
    exporter: SpanExporter = None,          # Custom span exporter (OTLP, Console, etc.)
    metrics_exporter: MetricExporter = None,# Custom metrics exporter
    logging_exporter: LogExporter = None,   # Custom logging exporter

    # === Processing ===
    disable_batch: bool = False,            # Use SimpleSpanProcessor (default: BatchSpanProcessor)
    processor: Union[SpanProcessor, List[SpanProcessor]] = None,  # Custom processor(s)

    # === Sampling ===
    sampler: Sampler = None,                # Custom sampler (default: AlwaysOn)

    # === Context Propagation ===
    propagator: TextMapPropagator = None,   # Custom propagator (default: W3C TraceContext)

    # === Instrumentation ===
    instruments: Set[Instruments] = None,   # Specific instruments to enable
    block_instruments: Set[Instruments] = None,  # Instruments to block

    # === Advanced ===
    headers: Dict[str, str] = {},           # Custom headers for exporter
    resource_attributes: dict = {},         # Custom resource attributes
    should_enrich_metrics: bool = True,     # Enhanced metrics with workflow context
    traceloop_sync_enabled: bool = False,   # Sync prompts/config with Traceloop
    image_uploader: ImageUploader = None,   # Custom image uploader for base64 images
    span_postprocess_callback: Callable = None,  # Post-process spans before export
)
```

### Environment Variables

```bash
# === Tracing ===
TRACELOOP_TRACING_ENABLED=true           # Enable/disable tracing (default: true)
TRACELOOP_TRACE_CONTENT=true             # Capture prompts/completions (default: true)

# === Metrics & Logging ===
TRACELOOP_METRICS_ENABLED=true           # Enable metrics collection (default: true)
TRACELOOP_LOGGING_ENABLED=false          # Enable logging collection (default: false)

# === Traceloop Cloud ===
TRACELOOP_API_KEY=<your-key>            # API key for Traceloop service
TRACELOOP_BASE_URL=<custom-endpoint>    # Custom Traceloop endpoint

# === Exporters ===
TRACELOOP_HEADERS='{"x-custom":"value"}'  # Custom headers (JSON)
TRACELOOP_METRICS_ENDPOINT=<url>         # Separate metrics endpoint
TRACELOOP_LOGGING_ENDPOINT=<url>         # Separate logging endpoint

# === Telemetry ===
TRACELOOP_TELEMETRY=true                # Anonymous telemetry (default: true)
```

### Privacy Controls

#### Disable Content Tracing

```bash
# Environment variable (recommended for production)
export TRACELOOP_TRACE_CONTENT=false
```

```python
# Or configure at init (not yet supported, use env var)
```

When disabled:
- ❌ Prompts/messages NOT captured
- ❌ Completions/responses NOT captured
- ❌ Tool arguments NOT captured
- ✅ Model names, token counts, durations STILL captured
- ✅ Span hierarchy and workflow structure STILL captured

#### Custom Content Filtering

For fine-grained control, use the `span_postprocess_callback`:

```python
def redact_sensitive_data(span):
    """Remove PII from spans before export"""
    for attribute_key in list(span.attributes.keys()):
        if 'email' in attribute_key or 'ssn' in attribute_key:
            span.set_attribute(attribute_key, "[REDACTED]")

Traceloop.init(
    app_name="my-app",
    span_postprocess_callback=redact_sensitive_data
)
```

---

## Manual Instrumentation

Auto-instrumentation covers individual LLM/DB calls, but you'll want to add **manual instrumentation** for:

- Multi-step workflows
- Business logic
- Custom functions
- Tool execution
- Application-specific context

### Decorator API

The Traceloop SDK provides four decorators for semantic instrumentation:

#### `@workflow` - Top-Level Entry Points

```python
from traceloop.sdk.decorators import workflow

@workflow(name="customer_support_bot")
def handle_support_request(user_query: str):
    """Top-level workflow - one per user request"""
    category = classify_intent(user_query)
    response = generate_response(user_query, category)
    return response

# Creates span: "customer_support_bot.workflow"
```

**Sample Reference**: [`packages/sample-app/sample_app/methods_decorated_app.py:L13-19`](../packages/sample-app/sample_app/methods_decorated_app.py)

#### `@task` - Individual Steps

```python
from traceloop.sdk.decorators import task

@task(name="classify_intent")
def classify_intent(query: str) -> str:
    """Task within a workflow"""
    # OpenAI call auto-instrumented
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": f"Classify: {query}"}]
    )
    return response.choices[0].message.content

# Creates span: "classify_intent.task"
# Child spans: "openai.chat" (auto-instrumented)
```

#### `@agent` - Agent Operations

```python
from traceloop.sdk.decorators import agent

@agent(name="research_agent")
def research_topic(topic: str):
    """Agent that performs research"""
    search_results = search_web(topic)
    summary = summarize_results(search_results)
    return summary

# Creates span: "research_agent.agent"
```

**Sample Reference**: [`packages/sample-app/sample_app/classes_decorated_app.py`](../packages/sample-app/sample_app/classes_decorated_app.py)

#### `@tool` - Tool/Function Calls

```python
from traceloop.sdk.decorators import tool

@tool(name="web_search")
def search_web(query: str) -> List[str]:
    """Tool for web search"""
    # External API call
    results = requests.get(f"https://api.search.com?q={query}")
    return results.json()

# Creates span: "web_search.tool"
```

### Decorator Features

All decorators support:

#### Sync and Async Functions

```python
@workflow(name="async_workflow")
async def async_workflow(query: str):
    result = await async_llm_call(query)
    return result
```

#### Input/Output Capture

```python
@task(name="summarize", version=2)
def summarize(text: str) -> str:
    result = llm_call(text)
    return result

# Span attributes (when TRACELOOP_TRACE_CONTENT=true):
# - traceloop.entity.input = '{"text": "..."}'
# - traceloop.entity.output = '{"result": "..."}'
```

#### Class Methods

```python
class AgentExecutor:
    @agent(name="executor", method_name="run")
    def run(self, task: str):
        return self.execute(task)

# Creates span: "executor.agent"
```

**Sample Reference**: [`packages/sample-app/sample_app/classes_decorated_app.py:L19-44`](../packages/sample-app/sample_app/classes_decorated_app.py)

#### Generators

```python
@task(name="stream_processor")
def process_stream(items):
    for item in items:
        yield process_item(item)

# Span created when generator is exhausted
```

### Span Hierarchy Example

```python
@workflow(name="rag_pipeline")
def rag_pipeline(question: str):
    # Workflow span created

    embedding = generate_embedding(question)  # Task span
    results = search_db(embedding)            # Task span with DB span child
    answer = generate_answer(question, results)  # Task span with LLM span child

    return answer

@task(name="generate_embedding")
def generate_embedding(text: str):
    # OpenAI embedding call auto-instrumented
    return client.embeddings.create(model="text-embedding-3-small", input=text)

@task(name="search_db")
def search_db(embedding):
    # Pinecone query auto-instrumented
    return index.query(vector=embedding, top_k=5)

@task(name="generate_answer")
def generate_answer(question: str, context):
    # OpenAI chat call auto-instrumented
    return client.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": f"{question}\n\nContext: {context}"}]
    )
```

**Resulting Span Hierarchy:**
```
rag_pipeline.workflow
├── generate_embedding.task
│   └── openai.embeddings (auto)
├── search_db.task
│   └── pinecone.query (auto)
└── generate_answer.task
    └── openai.chat (auto)
```

**Sample Reference**: [`packages/sample-app/sample_app/methods_decorated_app.py`](../packages/sample-app/sample_app/methods_decorated_app.py)

### Association Properties

Add metadata to ALL spans in the current trace context:

```python
from traceloop.sdk import Traceloop

@workflow(name="chat")
def handle_chat(user_id: str, session_id: str, message: str):
    # Set properties that will be added to ALL child spans
    Traceloop.set_association_properties({
        "user_id": user_id,
        "session_id": session_id,
        "environment": "production"
    })

    response = process_message(message)
    return response

# All spans in this trace will have:
# - association.properties.user_id
# - association.properties.session_id
# - association.properties.environment
```

This is useful for:
- User tracking
- Session management
- A/B testing
- Environment tagging
- Custom business context

### Manual Span Tracking

For advanced use cases, use the `track_llm_call` context manager:

```python
from traceloop.sdk.tracing.manual import track_llm_call, LLMMessage, LLMUsage

with track_llm_call(vendor="custom_llm", type="chat") as span:
    # Set request attributes
    span.report_request(
        model="my-model-v1",
        messages=[
            LLMMessage(role="user", content="Hello")
        ]
    )

    # Make your custom LLM call
    response = my_custom_llm_client.chat(...)

    # Set response attributes
    span.report_response(
        model="my-model-v1",
        response_messages=[
            LLMMessage(role="assistant", content=response.text)
        ]
    )

    # Set token usage
    span.report_usage(
        LLMUsage(
            input_tokens=response.usage.prompt_tokens,
            output_tokens=response.usage.completion_tokens,
            total_tokens=response.usage.total_tokens
        )
    )
```

**Sample Reference**: [`packages/sample-app/sample_app/manual_logging_example.py`](../packages/sample-app/sample_app/manual_logging_example.py)

---

## Advanced Features

### Prompt Registry

Manage prompts centrally with versioning and variables:

```python
from traceloop.sdk.prompts import get_prompt

# Fetch prompt from registry
prompt_args = get_prompt(
    key="joke_generator",
    variables={"persona": "pirate", "topic": "coding"}
)

# Use with OpenAI (prompt_args has model, messages, etc.)
response = client.chat.completions.create(**prompt_args)
```

**Sample Reference**: [`packages/sample-app/sample_app/prompt_registry_example_app.py`](../packages/sample-app/sample_app/prompt_registry_example_app.py)

### Datasets & Experiments

Track evaluation runs across datasets:

```python
from traceloop.sdk import Traceloop

client = Traceloop.get()

# Create dataset
dataset_id = client.datasets.create(
    name="customer_support_evals",
    description="Evaluation dataset for support bot"
)

# Add data items
client.datasets.add_items(
    dataset_id=dataset_id,
    items=[
        {"input": "How do I reset password?", "expected_output": "..."},
        {"input": "What's your refund policy?", "expected_output": "..."},
    ]
)

# Run experiment
for item in dataset_items:
    result = my_workflow(item["input"])
    # Results automatically tracked with dataset association
```

**Sample Reference**: [`packages/sample-app/sample_app/dataset_example.py`](../packages/sample-app/sample_app/dataset_example.py)

### User Feedback

Capture user feedback on AI outputs:

```python
from traceloop.sdk import Traceloop

client = Traceloop.get()

# After getting LLM response
response_span_id = "..."  # From trace context

# User provides feedback
client.user_feedback.create(
    task_id=response_span_id,
    user_id="user-123",
    feedback={
        "score": 0.9,
        "helpful": True,
        "comment": "Great response!"
    }
)
```

### Image Upload

For vision models, automatically upload base64 images to reduce trace size:

```python
async def upload_base64_image(trace_id, span_id, image_name, base64_data):
    """Upload image to cloud storage, return URL"""
    url = await upload_to_s3(base64_data)
    return url

Traceloop.init(
    app_name="vision-app",
    image_uploader=upload_base64_image
)

# Base64 images in prompts will be uploaded and replaced with URLs
```

### Multiple Span Processors

Use multiple processors simultaneously:

```python
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import (
    BatchSpanProcessor,
    ConsoleSpanExporter,
)
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter

# Console for debugging + OTLP for production
Traceloop.init(
    app_name="my-app",
    processor=[
        BatchSpanProcessor(ConsoleSpanExporter()),
        BatchSpanProcessor(OTLPSpanExporter(endpoint="http://localhost:4318/v1/traces"))
    ]
)
```

**Sample Reference**: [`packages/sample-app/sample_app/multiple_span_processors.py`](../packages/sample-app/sample_app/multiple_span_processors.py)

### Custom Sampling

Control which traces are exported:

```python
from opentelemetry.sdk.trace.sampling import ParentBasedTraceIdRatioBased

# Sample 10% of traces
Traceloop.init(
    app_name="my-app",
    sampler=ParentBasedTraceIdRatioBased(0.1)
)
```

Available samplers:
- `AlwaysOn` - Sample everything (default)
- `AlwaysOff` - Sample nothing
- `TraceIdRatioBased(ratio)` - Sample probabilistically
- `ParentBasedTraceIdRatioBased(ratio)` - Respect parent sampling decision
- Custom samplers

### Metrics Enrichment

Enrich metrics with workflow context:

```python
Traceloop.init(
    app_name="my-app",
    should_enrich_metrics=True  # Default
)

# Token usage metrics will include:
# - workflow.name
# - entity.name
# - Custom association properties
```

---

## Best Practices

### 1. Initialize Once at Startup

```python
# ✅ Good - in main.py or __init__.py
from traceloop.sdk import Traceloop

Traceloop.init(app_name="my-app")

# Import your modules AFTER initialization
from .routes import app

if __name__ == "__main__":
    app.run()
```

```python
# ❌ Bad - initializing multiple times
Traceloop.init(app_name="app1")
Traceloop.init(app_name="app2")  # This will cause issues
```

### 2. Use Decorators for Business Logic

```python
# ✅ Good - clear workflow structure
@workflow(name="process_document")
def process_document(doc_id: str):
    content = extract_text(doc_id)  # Task
    summary = summarize(content)     # Task with LLM call
    return summary

@task(name="extract_text")
def extract_text(doc_id: str):
    # Your logic
    return text
```

```python
# ❌ Less ideal - missing context
def process_document(doc_id: str):
    # No workflow context
    content = extract_text(doc_id)
    summary = summarize(content)
    return summary
```

### 3. Protect PII in Production

```bash
# Set in production environment
export TRACELOOP_TRACE_CONTENT=false
```

Or use selective filtering:
```python
def redact_pii(span):
    # Custom redaction logic
    pass

Traceloop.init(
    app_name="prod-app",
    span_postprocess_callback=redact_pii
)
```

### 4. Use Association Properties for Context

```python
@workflow(name="api_handler")
def handle_request(request):
    Traceloop.set_association_properties({
        "user_id": request.user_id,
        "tenant_id": request.tenant_id,
        "request_id": request.id
    })

    # All child spans will have this context
    return process(request)
```

### 5. Batch Processing in Production

```python
# ✅ Good - default BatchSpanProcessor reduces overhead
Traceloop.init(
    app_name="prod-app",
    disable_batch=False  # Default
)
```

```python
# ⚠️ Only for debugging - high overhead
Traceloop.init(
    app_name="debug-app",
    disable_batch=True  # SimpleSpanProcessor
)
```

### 6. Selective Instrumentation for Performance

```python
# If you only use OpenAI and Pinecone, don't load all 29 instrumentors
Traceloop.init(
    app_name="my-app",
    instruments={Instruments.OPENAI, Instruments.PINECONE}
)
```

### 7. Flush Before Exit

```python
import atexit
from traceloop.sdk.tracing import TracerWrapper

# Automatic flush registered by SDK
# But for short-lived scripts, force flush:

def main():
    Traceloop.init(app_name="script")

    # Your work
    result = llm_call()

    # Force flush before exit
    TracerWrapper.flush()

if __name__ == "__main__":
    main()
```

### 8. Testing with Console Exporter

```python
# In tests/conftest.py
import pytest
from opentelemetry.sdk.trace.export import ConsoleSpanExporter
from traceloop.sdk import Traceloop

@pytest.fixture(scope="session", autouse=True)
def tracing():
    Traceloop.init(
        app_name="test",
        exporter=ConsoleSpanExporter()
    )
```

---

## Troubleshooting

### Missing Spans

**Symptom**: LLM calls not appearing in traces

**Causes & Solutions**:

1. **SDK not initialized**
   ```python
   # Must call before imports
   Traceloop.init(app_name="my-app")
   ```

2. **Library not installed**
   ```bash
   # Install the library being instrumented
   pip install openai  # For OpenAI instrumentation
   ```

3. **Instrumentation disabled**
   ```python
   # Check you haven't blocked it
   Traceloop.init(
       app_name="my-app",
       # Don't use block_instruments unless intentional
   )
   ```

4. **Exporter not configured**
   ```python
   # Verify exporter is working
   from opentelemetry.sdk.trace.export import ConsoleSpanExporter

   Traceloop.init(
       app_name="debug",
       exporter=ConsoleSpanExporter()  # Should see output
   )
   ```

### Incomplete Data

**Symptom**: Spans created but missing prompts/responses

**Solution**: Enable content tracing
```bash
export TRACELOOP_TRACE_CONTENT=true
```

### Context Detachment Errors

**Symptom**: Warnings about detaching non-attached context

**Cause**: Context management in async code or frameworks

**Solution**: Usually harmless and handled gracefully by the SDK. If persistent:

```python
# For framework-specific issues, check instrumentation compatibility
# LangChain, for example, handles this automatically
```

### High Memory Usage

**Symptom**: Memory grows during long-running processes

**Causes & Solutions**:

1. **Batch processor not flushing**
   ```python
   # Reduce batch size
   from opentelemetry.sdk.trace.export import BatchSpanProcessor

   processor = BatchSpanProcessor(
       exporter,
       max_queue_size=512,      # Default: 2048
       schedule_delay_millis=5000,  # Default: 5000
       max_export_batch_size=128    # Default: 512
   )

   Traceloop.init(app_name="my-app", processor=processor)
   ```

2. **Streaming not properly closed**
   ```python
   # Always consume streams fully
   response = client.chat.completions.create(stream=True, ...)

   for chunk in response:
       # Process chunk
       pass
   # ✅ Stream fully consumed, span will close
   ```

### Performance Overhead

**Symptom**: Noticeable latency added by instrumentation

**Solutions**:

1. **Disable content tracing in production**
   ```bash
   export TRACELOOP_TRACE_CONTENT=false
   ```

2. **Use sampling**
   ```python
   from opentelemetry.sdk.trace.sampling import TraceIdRatioBased

   # Sample only 10%
   Traceloop.init(
       app_name="my-app",
       sampler=TraceIdRatioBased(0.1)
   )
   ```

3. **Selective instrumentation**
   ```python
   # Only instrument what you need
   Traceloop.init(
       app_name="my-app",
       instruments={Instruments.OPENAI}  # Skip others
   )
   ```

### Traces Not Appearing in Backend

**Symptom**: SDK running but no traces in Jaeger/Tempo/etc.

**Debugging Steps**:

1. **Test with console exporter**
   ```python
   from opentelemetry.sdk.trace.export import ConsoleSpanExporter

   Traceloop.init(
       app_name="debug",
       exporter=ConsoleSpanExporter()
   )
   # If you see JSON output, SDK is working
   ```

2. **Check exporter endpoint**
   ```python
   # Verify endpoint is correct
   from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter

   exporter = OTLPSpanExporter(
       endpoint="http://localhost:4318/v1/traces",  # Check this URL
       timeout=10  # Increase timeout if slow
   )
   ```

3. **Check network connectivity**
   ```bash
   # Test backend is reachable
   curl http://localhost:4318/v1/traces
   ```

4. **Enable debug logging**
   ```python
   import logging

   logging.basicConfig(level=logging.DEBUG)
   # OpenTelemetry will log export attempts
   ```

### Import Order Issues

**Symptom**: Libraries imported before `Traceloop.init()` not instrumented

**Solution**: Always initialize before imports

```python
# ✅ Correct order
from traceloop.sdk import Traceloop

Traceloop.init(app_name="my-app")

# Import AFTER initialization
from openai import OpenAI
from langchain import ...
```

```python
# ❌ Wrong order - won't be instrumented
from openai import OpenAI

from traceloop.sdk import Traceloop
Traceloop.init(app_name="my-app")  # Too late!
```

---

## Next Steps

- **Explore Instrumentation Packages**: See [`instrumentation/`](instrumentation/) for detailed docs on each provider/framework
- **View Sample Applications**: Check [`examples-index.md`](examples-index.md) for 64 working examples
- **Learn Patterns**: Read [`patterns/`](patterns/) for best practices and common use cases
- **Integration Guides**: See provider-specific docs for advanced features:
  - [OpenAI](instrumentation/llm-providers/openai.md) - Streaming, vision, tools, structured outputs
  - [Anthropic](instrumentation/llm-providers/anthropic.md) - Claude, extended thinking, prompt caching
  - [LangChain](instrumentation/frameworks/langchain.md) - LCEL, LangGraph, agents
  - More coming in future sessions

---

## Resources

- **OpenTelemetry**: https://opentelemetry.io/
- **GenAI Semantic Conventions**: https://opentelemetry.io/docs/specs/semconv/gen-ai/
- **Traceloop Docs**: https://traceloop.com/docs
- **GitHub Issues**: https://github.com/traceloop/openllmetry/issues

---

**Documentation Status**: ✅ Complete (Session 1)
**Last Updated**: 2025-11-22
**Next Review**: When new SDK features are added
