# Ollama Instrumentation

> **Package**: `opentelemetry-instrumentation-ollama`
> **Supported Versions**: ollama >= 0.4.0, < 1
> **Auto-Instrumentation**: ✅ Yes (via Traceloop SDK)
> **Provider**: Ollama (local models)

---

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [What's Auto-Instrumented](#whats-auto-instrumented)
- [What's Captured](#whats-captured)
- [Advanced Features](#advanced-features)
- [Configuration Options](#configuration-options)
- [Manual Instrumentation Scenarios](#manual-instrumentation-scenarios)
- [Sample Applications](#sample-applications)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)

---

## Overview

The Ollama instrumentation package automatically traces all Ollama API calls for local model inference. It supports chat, completions, and embeddings with full streaming support and tool calling capabilities.

**Key Features**:
- ✅ Complete API coverage (chat, generate, embeddings)
- ✅ Streaming support with time-to-first-token metrics
- ✅ Tool calling / function execution
- ✅ Async operations
- ✅ Works with all Ollama models (llama3, gemma, mistral, custom, etc.)
- ✅ Token usage tracking
- ✅ Local-first (no external API calls)

---

## Quick Start

### Step 1: Install and Run Ollama

```bash
# Install Ollama (macOS/Linux)
curl https://ollama.ai/install.sh | sh

# Pull a model
ollama pull llama3

# Verify it's running
ollama list
```

### Step 2: Use with Traceloop SDK (Recommended)

```python
from ollama import chat
from traceloop.sdk import Traceloop

# Initialize Traceloop - automatically instruments Ollama
Traceloop.init(app_name="my-ollama-app")

# Use Ollama as normal - all calls automatically traced
response = chat(
    model="llama3",
    messages=[{"role": "user", "content": "What is OpenTelemetry?"}]
)

print(response['message']['content'])
```

### Manual Instrumentation

```python
from ollama import chat
from opentelemetry.instrumentation.ollama import OllamaInstrumentor

# Initialize instrumentation
OllamaInstrumentor().instrument()

# Use Ollama - all calls are traced
response = chat(
    model="llama3",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

---

## What's Auto-Instrumented

### Instrumented Methods

| Method | Span Name | Request Type | Streaming | Async |
|--------|-----------|--------------|-----------|-------|
| `ollama.chat()` | `ollama.chat` | chat | ✅ | ✅ |
| `ollama.generate()` | `ollama.completion` | completion | ✅ | ✅ |
| `ollama.embeddings()` | `ollama.embeddings` | embedding | ❌ | ✅ |

**Supports**:
- All Ollama models (llama3, gemma, mistral, qwen, custom models)
- Text generation and conversations
- Multi-turn chat
- Function/tool calling
- Streaming responses
- Async/await patterns

---

## What's Captured

### Spans

Each API call creates a span with:
- **Span Name**: `ollama.chat`, `ollama.completion`, or `ollama.embeddings`
- **Span Kind**: `CLIENT`
- **Duration**: Total request time
- **Status**: `OK` or `ERROR` with exception details

### Request Attributes (GenAI Semantic Conventions)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.system` | Provider name | `"Ollama"` |
| `gen_ai.request.model` | Model identifier | `"llama3"` |
| `llm.is_streaming` | Streaming flag | `true` |

### Prompt Messages (When `TRACELOOP_TRACE_CONTENT=true`)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.prompt.{i}.role` | Message role | `"user"`, `"assistant"`, `"system"` |
| `gen_ai.prompt.{i}.content` | Message text | `"Hello, llama!"` |
| `gen_ai.prompt.{i}.tool_call_id` | Tool call ID (for tool messages) | `"call_123"` |
| `gen_ai.prompt.{i}.tool_calls.{j}.id` | Tool call identifier | `"call_456"` |
| `gen_ai.prompt.{i}.tool_calls.{j}.name` | Function name | `"get_weather"` |
| `gen_ai.prompt.{i}.tool_calls.{j}.arguments` | Arguments (JSON) | `"{\"location\":\"SF\"}"` |

### Response Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.response.model` | Model used | `"llama3"` |
| `gen_ai.usage.input_tokens` | Prompt tokens | `25` |
| `gen_ai.usage.output_tokens` | Completion tokens | `50` |
| `llm.usage.total_tokens` | Total tokens | `75` |

### Completion Content (When `TRACELOOP_TRACE_CONTENT=true`)

| Attribute | Description | Example |
|-----------|-------------|---------|
| `gen_ai.completion.0.content` | Response text | `"Here's the answer..."` |
| `gen_ai.completion.0.role` | Response role | `"assistant"` |

### Tool Definitions

| Attribute | Description | Example |
|-----------|-------------|---------|
| `llm.request.functions.{i}.name` | Function name | `"calculator"` |
| `llm.request.functions.{i}.description` | Function description | `"Perform calculations"` |
| `llm.request.functions.{i}.parameters` | JSON schema | `"{\"type\":\"object\"...}"` |

### Metrics

**Histograms**:
- `llm.token.usage` - Token counts (input/output) with `gen_ai.token.type` label
- `llm.operation.duration` - Request duration (seconds)
- `gen_ai.server.time_to_first_token` - Time to first token (streaming)
- `llm.streaming.time_to_generate` - Generation time after first token

**Common Attributes**: All metrics include `gen_ai.system` and `gen_ai.response.model`.

---

## Advanced Features

### Streaming

Full streaming support with real-time metrics:

```python
from ollama import chat

stream = chat(
    model="llama3",
    messages=[{"role": "user", "content": "Tell me a story"}],
    stream=True  # Enable streaming
)

for chunk in stream:
    content = chunk['message']['content']
    print(content, end='', flush=True)

# Automatic tracking:
# - Time to first token
# - Generation time
# - Complete response
# - Token usage
```

**Async Streaming**:

```python
import asyncio
from ollama import AsyncClient

async def stream_chat():
    client = AsyncClient()

    stream = await client.chat(
        model="llama3",
        messages=[{"role": "user", "content": "Hello"}],
        stream=True
    )

    async for chunk in stream:
        print(chunk['message']['content'], end='', flush=True)

asyncio.run(stream_chat())
```

**What's Tracked**:
- ✅ Time to first chunk (`gen_ai.server.time_to_first_token`)
- ✅ Total generation time after first token
- ✅ Complete accumulated response
- ✅ Final token counts

### Tool Calling

Ollama supports tool calling (function execution):

```python
tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get current weather",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {"type": "string", "description": "City name"}
            },
            "required": ["location"]
        }
    }
}]

response = chat(
    model="llama3.1",  # Tool calling requires llama3.1+
    messages=[{"role": "user", "content": "What's the weather in SF?"}],
    tools=tools
)

# Check for tool calls
if response['message'].get('tool_calls'):
    for tool_call in response['message']['tool_calls']:
        print(f"Tool: {tool_call['function']['name']}")
        print(f"Args: {tool_call['function']['arguments']}")
```

**Captured**:
- Tool definitions (`llm.request.functions`)
- Tool calls with IDs, names, and arguments
- Tool response messages

### Local Model Support

Works with any Ollama model (no restrictions):

```python
# Official models
models = ["llama3", "llama3.1", "gemma", "mistral", "qwen", "phi3"]

for model in models:
    response = chat(
        model=model,
        messages=[{"role": "user", "content": "Hi!"}]
    )
    # All automatically traced with model name in attributes
```

**Custom Models**:

```python
# Works with custom Modelfiles
response = chat(
    model="my-custom-model",  # Your custom model
    messages=[{"role": "user", "content": "Hello"}]
)

# Captured with model name in span attributes
```

### Async Operations

Full async support for all operations:

```python
import asyncio
from ollama import AsyncClient

async def async_chat():
    client = AsyncClient()

    # Async chat
    response = await client.chat(
        model="llama3",
        messages=[{"role": "user", "content": "Hello"}]
    )
    return response['message']['content']

result = asyncio.run(async_chat())
```

**Supported Async Operations**:
- `await client.chat()` - Async chat
- `await client.generate()` - Async generation
- `await client.embeddings()` - Async embeddings
- `await client.chat(stream=True)` - Async streaming

### Multi-Turn Conversations

```python
messages = [
    {"role": "user", "content": "What is 2+2?"}
]

# First turn
response = chat(model="llama3", messages=messages)
messages.append({"role": "assistant", "content": response['message']['content']})

# Second turn
messages.append({"role": "user", "content": "Now multiply that by 3"})
response = chat(model="llama3", messages=messages)

# Each turn creates a separate span
# All messages captured in span attributes
```

---

## Configuration Options

### Instrumentor Parameters

```python
from opentelemetry.instrumentation.ollama import OllamaInstrumentor

OllamaInstrumentor().instrument(
    # Error handling - custom exception logger
    exception_logger=None,

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

### Custom Exception Logging

```python
def log_exception(exc):
    print(f"Ollama instrumentation error: {exc}")
    # Send to monitoring service

OllamaInstrumentor(exception_logger=log_exception).instrument()
```

---

## Manual Instrumentation Scenarios

### When Auto-Instrumentation Isn't Enough

1. **Custom Workflow Context**: Add application-specific metadata

```python
from opentelemetry import trace
from traceloop.sdk.decorators import workflow

@workflow(name="qa_assistant")
def answer_question(question, context):
    span = trace.get_current_span()
    span.set_attribute("question.length", len(question))
    span.set_attribute("context.chunks", len(context))

    messages = [
        {"role": "system", "content": "You are a helpful assistant"},
        {"role": "user", "content": f"Context: {context}\n\nQuestion: {question}"}
    ]

    response = chat(model="llama3", messages=messages)  # Auto-instrumented
    return response['message']['content']
```

2. **Multi-Step Workflows**: Use task decorators

```python
from traceloop.sdk.decorators import workflow, task

@task(name="local_reasoning")
def reason_with_llama(prompt):
    return chat(
        model="llama3",
        messages=[{"role": "user", "content": prompt}]
    )['message']['content']

@workflow(name="local_rag")
def local_rag_pipeline(query):
    # Embed query locally
    embedding = get_local_embedding(query)

    # Search local vector DB
    context = search_local_vectors(embedding)

    # Reason with Ollama (traced)
    answer = reason_with_llama(f"Context: {context}\nQuery: {query}")

    return answer
```

3. **Performance Tracking**: Monitor local inference performance

```python
from opentelemetry import trace

def monitored_generation(prompt, model="llama3"):
    span = trace.get_current_span()
    span.set_attribute("model.size", get_model_size(model))
    span.set_attribute("prompt.tokens_estimate", len(prompt.split()))

    response = generate(model=model, prompt=prompt)

    # Add custom metrics
    if 'eval_count' in response:
        tokens_per_second = response['eval_count'] / response['eval_duration'] * 1e9
        span.set_attribute("inference.tokens_per_second", tokens_per_second)

    return response
```

---

## Sample Applications

### Main Sample

**File**: `packages/sample-app/sample_app/ollama_streaming.py`

**Features**:
- Streaming chat with `gemma3:1b` model
- Real-time output
- Traceloop SDK initialization
- Complete automatic instrumentation

**Code**:
```python
from traceloop.sdk import Traceloop
from ollama import chat

# Initialize Traceloop
Traceloop.init(app_name="ollama_streaming_app")

# Streaming chat
stream_response = chat(
    model="gemma3:1b",
    messages=[{
        "role": "user",
        "content": "Tell a joke about opentelemetry"
    }],
    stream=True
)

# Print streamed chunks
for chunk in stream_response:
    if chunk.message and chunk.message.content:
        print(chunk.message.content, end="", flush=True)

# Complete response and metrics captured automatically
```

### Running the Sample

```bash
# Ensure Ollama is running
ollama serve

# Pull the model
ollama pull gemma3:1b

# Install dependencies
cd packages/sample-app
poetry install

# Run sample
poetry run python sample_app/ollama_streaming.py
```

---

## Common Patterns

### Pattern 1: Local RAG System

```python
from traceloop.sdk.decorators import workflow, task
from ollama import chat, embeddings

@task(name="embed_query")
def embed_query(text):
    result = embeddings(model="nomic-embed-text", prompt=text)
    return result['embedding']

@task(name="retrieve_context")
def retrieve_context(embedding):
    # Search local vector database
    return local_search(embedding, top_k=3)

@task(name="generate_answer")
def generate_answer(question, context):
    messages = [
        {"role": "system", "content": "Answer based on context."},
        {"role": "user", "content": f"Context: {context}\n\nQuestion: {question}"}
    ]

    response = chat(model="llama3", messages=messages)
    return response['message']['content']

@workflow(name="local_rag")
def local_rag(question):
    # All steps traced automatically
    embedding = embed_query(question)
    context = retrieve_context(embedding)
    answer = generate_answer(question, context)
    return answer
```

### Pattern 2: Model Comparison

```python
@workflow(name="model_comparison")
def compare_models(prompt):
    """Compare responses from different models."""
    models = ["llama3", "gemma", "mistral"]
    results = {}

    for model in models:
        response = chat(
            model=model,
            messages=[{"role": "user", "content": prompt}]
        )
        results[model] = response['message']['content']

    # Each model call creates separate span with model name
    return results
```

### Pattern 3: Streaming with Progress

```python
def stream_with_progress(prompt, model="llama3"):
    """Stream response with token counting."""
    token_count = 0

    stream = chat(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        stream=True
    )

    for chunk in stream:
        content = chunk['message']['content']
        if content:
            print(content, end='', flush=True)
            token_count += len(content.split())

    print(f"\n\nGenerated ~{token_count} tokens")
```

### Pattern 4: Tool Calling Loop

```python
def chat_with_tools(user_input, tools):
    """Multi-turn chat with tool execution."""
    messages = [{"role": "user", "content": user_input}]

    while True:
        response = chat(model="llama3.1", messages=messages, tools=tools)

        # Add assistant response
        messages.append(response['message'])

        # Check for tool calls
        if not response['message'].get('tool_calls'):
            return response['message']['content']

        # Execute tools
        for tool_call in response['message']['tool_calls']:
            result = execute_tool(
                tool_call['function']['name'],
                tool_call['function']['arguments']
            )

            # Add tool result
            messages.append({
                "role": "tool",
                "content": result,
                "tool_call_id": tool_call['id']
            })
```

### Pattern 5: Async Batch Processing

```python
import asyncio
from ollama import AsyncClient

async def process_batch(prompts, model="llama3"):
    """Process multiple prompts in parallel."""
    client = AsyncClient()

    tasks = [
        client.chat(
            model=model,
            messages=[{"role": "user", "content": p}]
        )
        for p in prompts
    ]

    # All requests traced individually
    results = await asyncio.gather(*tasks)
    return [r['message']['content'] for r in results]
```

---

## Troubleshooting

### Issue: No spans appearing

**Symptoms**: Ollama works but no traces visible

**Solutions**:
1. Verify initialization order:
   ```python
   from traceloop.sdk import Traceloop
   Traceloop.init()  # Must be called first

   from ollama import chat  # Then import Ollama
   ```

2. Verify Ollama is running:
   ```bash
   ollama list  # Should show available models
   ```

3. Check exporter:
   ```python
   from opentelemetry.sdk.trace.export import ConsoleSpanExporter
   Traceloop.init(exporter=ConsoleSpanExporter())
   ```

### Issue: Token counts missing

**Symptoms**: Spans exist but token usage attributes empty

**Cause**: Ollama sometimes doesn't return token counts (especially for async).

**Solutions**:
- Use synchronous client when token counts are critical
- Check Ollama version (newer versions have better token reporting)
- Verify model supports token counting

### Issue: Content not captured

**Symptoms**: Spans exist but prompts/completions empty

**Solution**: Enable content tracing:
```bash
export TRACELOOP_TRACE_CONTENT=true
```

### Issue: Streaming metrics missing

**Symptoms**: Stream works but no time-to-first-token metric

**Cause**: Stream not fully consumed.

**Solution**:
```python
# Bad - stream abandoned
stream = chat(model="llama3", messages=messages, stream=True)
# Metrics not recorded!

# Good - consume stream
stream = chat(model="llama3", messages=messages, stream=True)
for chunk in stream:
    process(chunk)
# Metrics recorded after stream completes
```

### Issue: Ollama connection errors

**Symptoms**: Connection refused or timeout errors

**Solutions**:
1. Ensure Ollama is running:
   ```bash
   ollama serve
   ```

2. Check Ollama status:
   ```bash
   curl http://localhost:11434  # Should return "Ollama is running"
   ```

3. Verify model is downloaded:
   ```bash
   ollama list
   ollama pull llama3  # If not listed
   ```

### Issue: Slow inference performance

**Symptoms**: High latency in span durations

**Causes & Solutions**:
- **Large model**: Use smaller models (`gemma3:1b` vs `llama3:70b`)
- **CPU inference**: Check if GPU acceleration is available
- **Context length**: Reduce conversation history length
- **Concurrent requests**: Limit parallelism (Ollama queues requests)

**Monitor with traces**:
```python
# Check span durations and token counts
# Identify slow queries
# Optimize model selection based on use case
```

### Issue: Out of memory errors

**Symptoms**: Ollama crashes or returns memory errors

**Solutions**:
- Use smaller models
- Reduce context window size
- Limit concurrent requests
- Increase system memory allocation for Ollama

---

## Additional Resources

- **Ollama Documentation**: https://ollama.ai/
- **Ollama Python Library**: https://github.com/ollama/ollama-python
- **Ollama Models**: https://ollama.ai/library
- **OpenTelemetry GenAI Semantic Conventions**: https://opentelemetry.io/docs/specs/semconv/gen-ai/
- **Traceloop SDK Guide**: [../../traceloop-sdk-guide.md](../../traceloop-sdk-guide.md)
- **Sample Applications**: [../../examples-index.md](../../examples-index.md)

---

**Last Updated**: 2025-11-22 (Session 4)
