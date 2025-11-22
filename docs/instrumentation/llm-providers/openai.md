# OpenAI Instrumentation

> **Package**: `opentelemetry-instrumentation-openai`
> **Supported Versions**: OpenAI SDK >= 0.27.0
> **Auto-Instrumentation**: ✅ Yes (via Traceloop SDK)
> **Provider**: OpenAI, Azure OpenAI, and OpenAI-compatible providers

---

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [What's Auto-Instrumented](#whats-auto-instrumented)
- [What's Captured](#whats-captured)
- [Advanced Features](#advanced-features)
- [Azure OpenAI](#azure-openai)
- [Configuration Options](#configuration-options)
- [Manual Instrumentation Scenarios](#manual-instrumentation-scenarios)
- [Sample Applications](#sample-applications)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)

---

## Overview

The OpenAI instrumentation package automatically traces all OpenAI SDK API calls, capturing request parameters, response data, token usage, and performance metrics. It supports the full range of OpenAI APIs including chat completions, embeddings, assistants, and the new responses API.

**Key Features**:
- ✅ Complete API coverage (chat, completions, embeddings, assistants, responses)
- ✅ Streaming support with first-token timing
- ✅ Vision/multimodal with base64 image handling
- ✅ Function/tool calling instrumentation
- ✅ Structured outputs and Pydantic models
- ✅ Azure OpenAI integration
- ✅ Prompt caching detection
- ✅ Extended thinking (o1 models)
- ✅ Multi-provider support (AWS, Google, OpenRouter)

---

## Quick Start

### With Traceloop SDK (Recommended)

```python
from openai import OpenAI
from traceloop.sdk import Traceloop

# Initialize Traceloop - automatically instruments OpenAI
Traceloop.init(app_name="my-openai-app")

# Use OpenAI as normal - all calls are automatically traced
client = OpenAI()
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)
```

### Manual Instrumentation

```python
from openai import OpenAI
from opentelemetry.instrumentation.openai import OpenAIInstrumentor

# Initialize instrumentation
OpenAIInstrumentor().instrument()

# Use OpenAI - all calls are traced
client = OpenAI()
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

---

## What's Auto-Instrumented

All major OpenAI SDK methods are automatically instrumented:

### Chat Completions API

**Span Name**: `"openai.chat"`

| Method | Streaming | Async | Description |
|--------|-----------|-------|-------------|
| `client.chat.completions.create()` | ✅ | ✅ | Standard chat completions |
| `client.chat.completions.parse()` | ❌ | ✅ | Structured output parsing |
| `client.beta.chat.completions.parse()` | ❌ | ✅ | Beta structured parsing |

**Supports**:
- Text-only conversations
- Vision/multimodal inputs (images via URL or base64)
- Function/tool calling
- Structured outputs with `response_format`
- Streaming with `stream=True`
- Async variants with `AsyncOpenAI`

### Completions API

**Span Name**: `"openai.completion"`

| Method | Streaming | Async | Description |
|--------|-----------|-------|-------------|
| `client.completions.create()` | ✅ | ✅ | Legacy completions |

**Note**: Legacy API, prefer chat completions for modern applications.

### Embeddings API

**Span Name**: `"openai.embeddings"`

| Method | Async | Description |
|--------|-------|-------------|
| `client.embeddings.create()` | ✅ | Create text embeddings |

**Supports**:
- Single and batch text embedding
- All embedding models (`text-embedding-3-small`, `text-embedding-3-large`, `text-embedding-ada-002`)

### Assistants API (Beta)

**Span Name**: `"openai.assistant.run"`

| Method | Async | Description |
|--------|-------|-------------|
| `client.beta.assistants.create()` | ❌ | Create assistant |
| `client.beta.threads.runs.create()` | ❌ | Start a run |
| `client.beta.threads.runs.retrieve()` | ❌ | Get run status |
| `client.beta.threads.runs.create_and_stream()` | ❌ | Streaming run |
| `client.beta.threads.messages.list()` | ❌ | List messages |

**Supports**:
- Assistant lifecycle management
- Thread creation and messaging
- Tool execution (code interpreter, file search)
- Run status polling

### Responses API (Preview)

**Span Name**: `"openai.response"`

| Method | Async | Description |
|--------|-------|-------------|
| `client.responses.create()` | ✅ | Create response |
| `client.responses.retrieve()` | ✅ | Get response |
| `client.responses.cancel()` | ✅ | Cancel response |

### Images API

**Span Name**: `"openai.image"`

| Method | Description |
|--------|-------------|
| `client.images.generate()` | Generate images (DALL-E) |

**Note**: Metrics-only instrumentation (duration, exceptions).

---

## What's Captured

### Spans

Each API call creates a span with:
- **Span Name**: API-specific (e.g., `"openai.chat"`, `"openai.embeddings"`)
- **Span Kind**: `CLIENT`
- **Duration**: Total request time
- **Status**: `OK` or `ERROR` with exception details

### Request Attributes (GenAI Semantic Conventions)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.system` | Provider name | `"openai"`, `"Azure"` |
| `gen_ai.request.model` | Model identifier | `"gpt-4o-mini"` |
| `gen_ai.request.max_tokens` | Max tokens limit | `1000` |
| `gen_ai.request.temperature` | Temperature setting | `0.7` |
| `gen_ai.request.top_p` | Top-p sampling | `1.0` |
| `gen_ai.request.structured_output_schema` | JSON schema for response format | `{...}` |

### Response Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.response.model` | Actual model used | `"gpt-4o-mini-2024-07-18"` |
| `gen_ai.response.id` | Response ID | `"chatcmpl-..."` |
| `gen_ai.usage.input_tokens` | Prompt tokens | `150` |
| `gen_ai.usage.output_tokens` | Completion tokens | `300` |

### OpenAI-Specific Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| `llm.openai.api.base` | API endpoint | `"https://api.openai.com/v1"` |
| `llm.openai.api.version` | API version (Azure) | `"2024-08-01-preview"` |
| `llm.openai.response.system_fingerprint` | System fingerprint | `"fp_..."` |
| `llm.is_streaming` | Streaming flag | `true` |
| `llm.frequency_penalty` | Frequency penalty | `0.0` |
| `llm.presence_penalty` | Presence penalty | `0.0` |
| `llm.usage.total_tokens` | Total tokens | `450` |
| `llm.usage.cache_read_input_tokens` | Cached prompt tokens | `100` |
| `llm.usage.reasoning_tokens` | Reasoning tokens (o1 models) | `250` |
| `llm.request.reasoning_effort` | Reasoning effort level | `"medium"` |

### Message Content (When `TRACELOOP_TRACE_CONTENT=true`)

**Prompt Messages**:
- `gen_ai.prompt.{i}.role` - Message role (`"user"`, `"system"`, `"assistant"`)
- `gen_ai.prompt.{i}.content` - Message text

**Completion Messages**:
- `gen_ai.completion.{i}.role` - Response role
- `gen_ai.completion.{i}.content` - Response text
- `gen_ai.completion.{i}.refusal` - Safety refusal message (if any)
- `gen_ai.completion.{i}.reasoning` - Reasoning content (o1 models)

**Tool Calls**:
- `gen_ai.completion.{i}.tool_calls.{j}.id` - Tool call ID
- `gen_ai.completion.{i}.tool_calls.{j}.name` - Function name
- `gen_ai.completion.{i}.tool_calls.{j}.arguments` - JSON arguments

### Metrics

**Histograms**:
- `llm.token.usage` - Token counts with labels (input/output)
- `llm.operation.duration` - Operation duration in seconds
- `gen_ai.server.time_to_first_token` - Time to first token (streaming)
- `llm.streaming.time_to_generate` - Time between first and last token

**Counters**:
- `llm.chat.completions.choices` - Number of choices generated
- `llm.completions.exceptions` - Exception count by type
- `llm.embeddings.vector_size` - Embedding dimensions
- `llm.embeddings.exceptions` - Embedding API exceptions
- `llm.image_generations.exceptions` - Image generation exceptions

---

## Advanced Features

### Streaming

Both synchronous and asynchronous streaming are fully instrumented:

```python
from openai import OpenAI
from traceloop.sdk import Traceloop

Traceloop.init()
client = OpenAI()

# Streaming chat - automatically tracked
stream = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Tell me a story"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

**What's Tracked**:
- ✅ Time to first token (`gen_ai.server.time_to_first_token`)
- ✅ Time to generate complete response
- ✅ Complete accumulated response content
- ✅ Final token usage
- ✅ All response metadata

**Implementation**: The instrumentation wraps the stream object transparently, accumulating chunks while allowing normal iteration.

### Vision/Multimodal

Vision inputs are automatically instrumented with special handling for base64 images:

```python
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "What's in this image?"},
            {"type": "image_url", "image_url": {"url": "https://..."}}
        ]
    }]
)
```

**Base64 Image Handling**:

```python
from opentelemetry.instrumentation.openai import OpenAIInstrumentor

# Configure callback to upload base64 images externally
async def upload_image(trace_id, span_id, image_name, base64_data):
    # Upload to S3, Cloud Storage, etc.
    url = await upload_to_storage(base64_data)
    return url

OpenAIInstrumentor().instrument(
    upload_base64_image=upload_image
)
```

**Why?** Base64-encoded images can be very large. The callback allows you to upload them externally and replace the base64 data with a URL in traces.

**Sample**: See `packages/sample-app/sample_app/openai_vision_base64_example.py`

### Function/Tool Calling

Function and tool calls are fully traced with definitions and execution:

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"}
                }
            }
        }
    }
]

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "What's the weather in SF?"}],
    tools=tools
)

# Tool definitions captured in span attributes
# Tool calls captured in completion attributes
```

**Captured Attributes**:
- `llm.request.functions.{i}.name` - Tool name
- `llm.request.functions.{i}.description` - Tool description
- `llm.request.functions.{i}.parameters` - JSON schema
- `gen_ai.completion.{i}.tool_calls.{j}.id` - Call ID
- `gen_ai.completion.{i}.tool_calls.{j}.name` - Function invoked
- `gen_ai.completion.{i}.tool_calls.{j}.arguments` - Arguments

**Sample**: See `packages/sample-app/sample_app/openai_functions.py`

### Structured Outputs

Structured outputs with `response_format` and Pydantic models are instrumented:

```python
from pydantic import BaseModel

class Story(BaseModel):
    title: str
    characters: list[str]
    setting: str

response = client.beta.chat.completions.parse(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Write a short story"}],
    response_format=Story
)

story = response.choices[0].message.parsed
```

**Captured**: The JSON schema is extracted from the Pydantic model and stored in `gen_ai.request.structured_output_schema`.

**Sample**: See `packages/sample-app/sample_app/openai_structured_outputs.py`

### Extended Thinking (o1 Models)

Reasoning models (o1-preview, o1-mini) with extended thinking are instrumented:

```python
response = client.chat.completions.create(
    model="o1-preview",
    messages=[{"role": "user", "content": "Solve this complex problem..."}],
    reasoning_effort="medium"  # "low", "medium", "high"
)
```

**Captured**:
- `llm.request.reasoning_effort` - Reasoning level requested
- `llm.usage.reasoning_tokens` - Tokens used for thinking
- `gen_ai.completion.{i}.reasoning` - Reasoning content (if exposed)

**Requires**: OpenAI SDK >= 1.58.0

### Prompt Caching

Prompt caching is automatically detected and tracked:

```python
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "Very long system prompt..."},  # Cached
        {"role": "user", "content": "User query"}
    ]
)
```

**Captured**: `llm.usage.cache_read_input_tokens` shows how many tokens were served from cache.

**Benefits**:
- Reduced latency visibility
- Cost optimization tracking
- Cache hit rate analysis

### Assistants API

Full assistants workflow instrumentation with optional enrichment:

```python
from opentelemetry.instrumentation.openai import OpenAIInstrumentor

# Enable assistant enrichment
OpenAIInstrumentor().instrument(enrich_assistant=True)

client = OpenAI()

# Create assistant - tracked
assistant = client.beta.assistants.create(
    model="gpt-4o-mini",
    instructions="You are a helpful assistant"
)

# Create thread and run - all tracked
thread = client.beta.threads.create()
message = client.beta.threads.messages.create(
    thread_id=thread.id,
    role="user",
    content="Hello!"
)

run = client.beta.threads.runs.create(
    thread_id=thread.id,
    assistant_id=assistant.id
)

# Polling and completion tracked
while run.status in ["queued", "in_progress"]:
    run = client.beta.threads.runs.retrieve(thread_id=thread.id, run_id=run.id)
```

**With `enrich_assistant=True`**:
- Assistant instructions included in span
- Assistant metadata captured
- Thread details enriched

**Sample**: See `packages/sample-app/sample_app/openai_assistant.py`

---

## Azure OpenAI

Azure OpenAI is fully supported with automatic detection and configuration:

### Setup

```python
from openai import AzureOpenAI
from traceloop.sdk import Traceloop

Traceloop.init()

client = AzureOpenAI(
    api_key="your-azure-key",
    api_version="2024-08-01-preview",
    azure_endpoint="https://your-resource.openai.azure.com/"
)

# Use normally - automatically traced as Azure
response = client.chat.completions.create(
    model="gpt-4o-mini",  # Deployment name
    messages=[{"role": "user", "content": "Hello!"}]
)
```

### What's Different

**Attributes Captured**:
- `gen_ai.system` = `"Azure"`
- `llm.openai.api.version` = `"2024-08-01-preview"`
- `llm.openai.api.type` = `"azure"`
- `llm.openai.api.base` = Your Azure endpoint

**Model Names**: Use your deployment name (e.g., `"gpt-4o-mini"`, `"my-gpt4-deployment"`)

**All Features Supported**:
- ✅ Chat completions
- ✅ Embeddings
- ✅ Streaming
- ✅ Vision
- ✅ Function calling
- ✅ Structured outputs

**Sample**: See `packages/sample-app/sample_app/azure_openai.py`

---

## Configuration Options

### Instrumentor Parameters

```python
from opentelemetry.instrumentation.openai import OpenAIInstrumentor

OpenAIInstrumentor().instrument(
    # Assistant enrichment - include assistant details in traces
    enrich_assistant=False,

    # Error handling - custom exception logger
    exception_logger=None,

    # Custom metric attributes - add custom dimensions to metrics
    get_common_metrics_attributes=lambda: {"env": "production"},

    # Image handling - upload base64 images externally
    upload_base64_image=my_upload_function,

    # Trace context propagation - W3C trace context in API headers
    enable_trace_context_propagation=True,

    # Legacy vs events mode - use span attributes (True) vs events (False)
    use_legacy_attributes=True
)
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `TRACELOOP_TRACE_CONTENT` | `"true"` | Capture prompts and completions |
| `TRACELOOP_METRICS_ENABLED` | `"true"` | Enable metrics collection |

**Privacy Control**:

```bash
# Disable content capture for privacy
export TRACELOOP_TRACE_CONTENT=false
```

With content tracing disabled:
- ✅ Model names, token counts, durations still captured
- ❌ Actual prompt/completion text NOT captured
- ❌ Tool call arguments NOT captured

### Runtime Content Control

```python
from opentelemetry import context
from traceloop.sdk.tracing import set_association_properties

# Temporarily disable content tracing
with context.attach(context.set_value("override_enable_content_tracing", False)):
    response = client.chat.completions.create(...)  # No content captured
```

---

## Manual Instrumentation Scenarios

### When Auto-Instrumentation Isn't Enough

1. **Custom Span Attributes**: Add application-specific context

```python
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("user-query") as span:
    span.set_attribute("user.id", user_id)
    span.set_attribute("query.type", "support")

    response = client.chat.completions.create(...)  # Auto-instrumented
```

2. **Workflow Tracking**: Use Traceloop decorators for multi-step workflows

```python
from traceloop.sdk.decorators import workflow, task

@task(name="generate_response")
def generate_response(prompt):
    return client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}]
    )

@workflow(name="customer_support")
def handle_support_ticket(ticket):
    summary = generate_response(f"Summarize: {ticket}")
    response = generate_response(f"Respond to: {summary}")
    return response
```

3. **Custom Metrics**: Add business metrics alongside telemetry

```python
from opentelemetry import metrics

meter = metrics.get_meter(__name__)
query_counter = meter.create_counter("support.queries")

def handle_query(query):
    query_counter.add(1, {"type": query.type})
    return client.chat.completions.create(...)
```

---

## Sample Applications

All samples located in `packages/sample-app/sample_app/`

| File | Features Demonstrated |
|------|----------------------|
| `openai_vision_base64_example.py` | Vision API, base64 images, upload callback |
| `openai_functions.py` | Function/tool calling, multiple tools |
| `openai_assistant.py` | Assistants API, threads, runs, enrichment |
| `openai_streaming.py` | Streaming chat, workflow decorator |
| `openai_structured_outputs.py` | Structured outputs, Pydantic models, `parse()` |
| `openai_agents_example.py` | Agent patterns, multi-turn interactions |
| `openai_streaming_assistant.py` | Streaming from assistants |
| `azure_openai.py` | Azure OpenAI setup and usage |

### Running Samples

```bash
# Install dependencies
cd packages/sample-app
poetry install

# Set API key
export OPENAI_API_KEY=your-key-here

# Run a sample
poetry run python sample_app/openai_streaming.py
```

---

## Common Patterns

### 1. Multi-Turn Conversations

```python
from traceloop.sdk.decorators import workflow

@workflow(name="conversation")
def chat_conversation(messages):
    """Entire conversation tracked as one workflow"""
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=messages
    )
    return response
```

### 2. Retry Logic with Tracing

```python
from opentelemetry import trace
import time

def chat_with_retry(messages, max_retries=3):
    tracer = trace.get_tracer(__name__)

    for attempt in range(max_retries):
        try:
            with tracer.start_as_current_span(f"attempt-{attempt + 1}"):
                return client.chat.completions.create(
                    model="gpt-4o-mini",
                    messages=messages
                )
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(2 ** attempt)
```

### 3. Parallel Requests

```python
import asyncio
from openai import AsyncOpenAI

async def parallel_completions(prompts):
    client = AsyncOpenAI()

    tasks = [
        client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": p}]
        )
        for p in prompts
    ]

    # All requests traced individually
    return await asyncio.gather(*tasks)
```

### 4. Streaming with Accumulation

```python
def stream_and_save(prompt):
    """Stream response and save complete text"""
    stream = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        stream=True
    )

    complete_text = ""
    for chunk in stream:
        content = chunk.choices[0].delta.content
        if content:
            print(content, end="")
            complete_text += content

    return complete_text
```

---

## Troubleshooting

### Issue: No spans appearing

**Symptoms**: OpenAI calls work but no traces visible

**Solutions**:
1. Verify instrumentation is initialized:
   ```python
   from traceloop.sdk import Traceloop
   Traceloop.init()  # Must be called before OpenAI import
   ```

2. Check if instrumentation is suppressed:
   ```python
   from opentelemetry import context
   from opentelemetry.instrumentation.openai.shared import SUPPRESS_LANGUAGE_MODEL_INSTRUMENTATION_KEY

   # Ensure not suppressed
   assert context.get_value(SUPPRESS_LANGUAGE_MODEL_INSTRUMENTATION_KEY) != True
   ```

3. Verify exporter is configured:
   ```python
   from opentelemetry.sdk.trace.export import ConsoleSpanExporter
   Traceloop.init(exporter=ConsoleSpanExporter())  # Debug with console
   ```

### Issue: Content not captured

**Symptoms**: Spans exist but prompt/completion content is empty

**Solution**: Enable content tracing:
```bash
export TRACELOOP_TRACE_CONTENT=true
```

Or verify it's not disabled at runtime:
```python
from opentelemetry import context

# Check current setting
print(context.get_value("override_enable_content_tracing"))
```

### Issue: Streaming spans incomplete

**Symptoms**: Streaming responses show partial data

**Cause**: Stream must be fully consumed for span to complete.

**Solution**:
```python
# Bad - stream not consumed
stream = client.chat.completions.create(..., stream=True)
# Span still open!

# Good - consume stream
stream = client.chat.completions.create(..., stream=True)
for chunk in stream:
    process(chunk)
# Span now complete
```

### Issue: High cardinality metrics

**Symptoms**: Too many unique metric combinations

**Cause**: User IDs or other high-cardinality data in metric attributes.

**Solution**: Filter attributes in common metrics:
```python
OpenAIInstrumentor().instrument(
    get_common_metrics_attributes=lambda: {
        "env": "production",
        # Don't include user_id, session_id, etc.
    }
)
```

### Issue: Large base64 images in traces

**Symptoms**: Traces are very large due to embedded images

**Solution**: Configure upload callback:
```python
async def upload_to_s3(trace_id, span_id, image_name, base64_data):
    url = await s3_upload(base64_data)
    return url

OpenAIInstrumentor().instrument(upload_base64_image=upload_to_s3)
```

### Issue: Azure deployment names not showing

**Symptoms**: Model names missing for Azure

**Cause**: Using deployment IDs instead of names.

**Solution**: Ensure deployment name is used:
```python
# Bad
response = client.chat.completions.create(model="abc123")

# Good
response = client.chat.completions.create(model="gpt-4o-mini")
```

### Issue: Missing metrics

**Symptoms**: Spans appear but no metrics

**Solution**: Enable metrics:
```bash
export TRACELOOP_METRICS_ENABLED=true
```

And ensure meter provider is configured:
```python
from opentelemetry import metrics
from opentelemetry.sdk.metrics import MeterProvider

metrics.set_meter_provider(MeterProvider())
```

---

## Additional Resources

- **OpenAI SDK Documentation**: https://platform.openai.com/docs
- **Azure OpenAI Documentation**: https://learn.microsoft.com/en-us/azure/ai-services/openai/
- **OpenTelemetry GenAI Semantic Conventions**: https://opentelemetry.io/docs/specs/semconv/gen-ai/
- **Traceloop SDK Guide**: [../../traceloop-sdk-guide.md](../../traceloop-sdk-guide.md)
- **Sample Applications**: [../../examples-index.md](../../examples-index.md)

---

**Last Updated**: 2025-11-22 (Session 2)
